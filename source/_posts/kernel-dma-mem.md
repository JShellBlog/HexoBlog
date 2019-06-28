---
title: kernel_dma_mem
date: 2019-06-28 13:55:44
tags:
    - memory
    - kernel
categories:
    - memory
---

在Kernel 中对于DMA 的一致性主要在驱动中会有需求，因为提供给硬件的地址，必须是总线地址即硬件地址且是连续的。为此，kernel提供了两种方式的DMA 映射：
- 一致性DMA 映射（Coherent DMA Map）  
- 流式DMA 映射（Streaming DMA Map） 

<!--more-->

## 1. 一致性DMA 映射
### 1.1. 一致性DMA
kernel 提供了如下函数：
``` c

void *dma_alloc_coherent(struct device *dev, size_t size,
				       dma_addr_t *dma_handle, gfp_t flag,
				       struct dma_attrs *attrs);

#define dma_free_coherent(d, s, c, h) dma_free_attrs(d, s, c, h, NULL)

void dma_free_coherent(struct device *dev, size_t size,
				     void *cpu_addr, dma_addr_t dma_handle,
				     struct dma_attrs *attrs);
```

dma_alloc_coherent（）-> arm_dma_alloc()
```c
void *arm_dma_alloc(struct device *dev, size_t size, dma_addr_t *handle,
		    gfp_t gfp, struct dma_attrs *attrs)
{
	pgprot_t prot = __get_dma_pgprot(attrs, PAGE_KERNEL);
	void *memory;

	if (dma_alloc_from_coherent(dev, size, handle, &memory))
		return memory;

	return __dma_alloc(dev, size, handle, gfp, prot, false,
			   __builtin_return_address(0));
}

static void *__dma_alloc(struct device *dev, size_t size, dma_addr_t *handle,
			 gfp_t gfp, pgprot_t prot, bool is_coherent, const void *caller)
{
    ...
    if (is_coherent || nommu())
		addr = __alloc_simple_buffer(dev, size, gfp, &page);
	else if (!(gfp & __GFP_WAIT))
		addr = __alloc_from_pool(size, &page);
	else if (!dev_get_cma_area(dev))
		addr = __alloc_remap_buffer(dev, size, gfp, prot, &page, caller);
	else
		addr = __alloc_from_contiguous(dev, size, prot, &page, caller);
    
}
```

根据配置，我们选择__alloc_remap_buffer（） 函数进行alloc， CMA 的方式了解请看下面的参看资料中的连接。[CMA模块学习笔记](http://www.wowotech.net/memory_management/cma.html)

在__alloc_remap_buffer（） 会进行alloc_pages 操作，并在__dma_alloc_remap（）函数中进行page 属性的重新设定，标示为**nocache**, 这样就保证了CPU 与MEM 之间数据的一致性。
```c
static void *__alloc_remap_buffer(struct device *dev, size_t size, gfp_t gfp,
				 pgprot_t prot, struct page **ret_page,
				 const void *caller)
{
	struct page *page;
	void *ptr;
	page = __dma_alloc_buffer(dev, size, gfp);
	if (!page)
		return NULL;

	ptr = __dma_alloc_remap(page, size, gfp, prot, caller);
	if (!ptr) {
		__dma_free_buffer(page, size);
		return NULL;
	}

	*ret_page = page;
	return ptr;
}

static void *
__dma_alloc_remap(struct page *page, size_t size, gfp_t gfp, pgprot_t prot,
	const void *caller)
{
	/*
	 * DMA allocation can be mapped to user space, so lets
	 * set VM_USERMAP flags too.
	 */
	return dma_common_contiguous_remap(page, size,
			VM_ARM_DMA_CONSISTENT | VM_USERMAP,
			prot, caller);
}
```


### 1.2 DMA 池
```c
dma_pool_create(name, dev, size, align, alloc);
dma_pool_destroy(pool) pci_pool_destroy(pool);
dma_pool_alloc(pool, flags, handle) pci_pool_alloc(pool, flags, handle);
dma_pool_free(pool, vaddr, addr) pci_pool_free(pool, vaddr, addr);
```

## 2. 流式DMA 映射
流式映射，kernel 还分为了两种形式：
- 多page 
- 单个page 

### 2.1. 多个page 流式映射
LDD3 上也叫做分散/聚集映射，原理大致与单个page 映射相同， 不同点在于多个循环映射 __struct scatterlist *sg__ 里面聚集的pages 

dma_data_direction分为如下几种：
> - DMA_BIDIRECTIONAL （双向）
> - DMA_FROM_DEVICE  
> - DMA_TO_DEVICE  
> - DMA_NONE 

```c
enum dma_data_direction {
	DMA_BIDIRECTIONAL = 0,
	DMA_TO_DEVICE = 1,
	DMA_FROM_DEVICE = 2,
	DMA_NONE = 3,
};
```

```c
int dma_map_sg(struct device *dev, struct scatterlist *sg,
				   int nents, enum dma_data_direction dir,
				   struct dma_attrs *attrs);

void dma_unmap_sg(struct device *dev, struct scatterlist *sg,
				      int nents, enum dma_data_direction dir,
				      struct dma_attrs *attrs);
```

**注意：在dma_unmap_sg 之前，CPU 是不能操作page的， 原因在于这些addr 都是cached 地址，只有在保证地址被flush 到主存中才能让CPU 操作**

Kernel 封装了如下函数，帮助我们显示的刷新cache 的数据，保证数据cache, memory 之间的一致性。

使用dma_sync_sg_for_cpu（）函数，将使用权给CPU（之后Device 不能访问，否则数据不一致），
使用dma_sync_sg_for_device（）函数，将使用权给Device（CPU 不能访问）。

```c
void
dma_sync_sg_for_cpu(struct device *dev, struct scatterlist *sg,
		    int nelems, enum dma_data_direction dir);

void
dma_sync_sg_for_device(struct device *dev, struct scatterlist *sg,
		       int nelems, enum dma_data_direction dir)  ;
```
究其原因，是因为流式DMA缓冲区是cached，在map时刷了下cache，在设备DMA完成unmap时再刷cache（根据数据流向写回或者无效），来保证了cache数据一致性，在unmap之前CPU操作缓冲区是不能保证数据一致的。因此kernel需要严格保证操作时序。
当然kernel也提供函数dma_sync_sg_for_cpu与dma_sync_sg_for_device，可以在未释放时操作缓冲区，很明显这2个函数实现中肯定是再次进行刷新cache的操作保证数据一致性。

### 2.2. 单个page 流式映射
```c
dma_map_single(dev, addr, size, dir);
dma_unmap_single(dev, hnd, size, dir);
```

与前面分散/聚集的多页映射一样， 我们不能在Device 访问期间，CPU 在访问，两者不能交互访问。
```c
void dma_sync_single_for_cpu(struct device *dev, dma_addr_t addr,
					   size_t size,
					   enum dma_data_direction dir);

void dma_sync_single_for_device(struct device *dev,
					      dma_addr_t addr, size_t size,
					      enum dma_data_direction dir); 
```                       

dma_map_single ==> __dma_map_page ==> __dma_page_cpu_to_dev ==> ___dma_page_cpu_to_dev 
```c
static void __dma_page_cpu_to_dev(struct page *page, unsigned long off,
	size_t size, enum dma_data_direction dir)
{
	phys_addr_t paddr;

	dma_cache_maint_page(page, off, size, dir, dmac_map_area);

	paddr = page_to_phys(page) + off;
	if (dir == DMA_FROM_DEVICE) {
		outer_inv_range(paddr, paddr + size);
	} else {
		outer_clean_range(paddr, paddr + size);
	}
	/* FIXME: non-speculating: flush on bidirectional mappings? */
}
```

dmac_map_area 在我们配置的ARMV7 是指向的rch/arm/mm/cache-v7.S 文件中
```asm
ENTRY(v7_dma_map_area)
	add	r1, r1, r0
	teq	r2, #DMA_FROM_DEVICE
	beq	v7_dma_inv_range
	b	v7_dma_clean_range
ENDPROC(v7_dma_map_area)

ENTRY(v7_dma_unmap_area)
	add	r1, r1, r0
	teq	r2, #DMA_TO_DEVICE
	bne	v7_dma_inv_range
	ret	lr
ENDPROC(v7_dma_unmap_area)
```
我们由此可以看出dma_map_sigle() 最终是使用汇编指令clean 或者invalidate cache 里面的数据，保证cache 与mem 之间的一致性。因此，在给Device 访问后，禁止CPU 访问，除非使用dma_sync_single_for_cpu（）函数，以此来将cache 数据invalidate。

## 参看资料
https://blog.csdn.net/skyflying2012/article/
https://my.oschina.net/yepanl/blog/3053881
http://www.wowotech.net/memory_management/cma.html