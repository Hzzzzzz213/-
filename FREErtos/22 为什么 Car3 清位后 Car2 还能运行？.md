

因为执行顺序不是：

> Car3 运行 → 清除 bit0 → Car2 再检查 bit0 → 发现没有 bit0，继续阻塞

实际更接近：

> Car1 调用 `xEventGroupSetBits(g_xEventCar, 1<<0)`  
> FreeRTOS 检查事件组等待列表  
> 对满足条件的等待任务解除阻塞，让它们进入就绪态  
> 调度器选择最高优先级就绪任务运行  
> Car3 优先级更高，先运行  
> Car3 从 `xEventGroupWaitBits()` 返回前清除 bit0  
> 但 Car2 如果已经在前面被解除阻塞，它已经是就绪态，不会因为 bit0 被清而重新阻塞  
> 所以 Car2 仍然能从 `xEventGroupWaitBits()` 返回并继续运行

也就是说：

**Car2 能运行，不是因为 bit0 还在，而是因为它在 bit0 被清掉之前已经“等到”了事件，已经从等待中出来了。**

`xClearOnExit = pdTRUE` 只清除事件组中的事件位，不会把已经因事件满足而解除阻塞的任务重新阻塞。  
==Car2 还能运行，是因为它在 Car3 清 bit0 之前已经等到事件、进入就绪态；==Car3 后面清 bit0 不影响它继续从 `xEventGroupWaitBits()` 返回。  
但多个任务等同一个 bit 且都用 clear-on-exit 有竞争，不能保证两个任务都可靠运行。