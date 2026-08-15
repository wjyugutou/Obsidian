```java

// 任何异常都回滚
@Transactional
public void transfer() {
    try {
        doTransfer();
    } catch (RuntimeException e) {
        log.error("转账失败", e);
        // ❌ 异常被吞了，Spring 看不到异常 → 不回滚！
    }
}
```

如果必须 catch，请重新抛出或使用 `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()` 手动标记回滚。

💡 **为什么推荐 `rollbackFor = Exception.class`？**  
因为实际项目中，很多第三方库（如 MyBatis、JPA）会抛出 Checked Exception，如果漏掉回滚会导致数据不一致。阿里 Java 开发手册也强制要求这样做。

**setRollbackOnly 示例**
**90% 的情况你不需要 `setRollbackOnly()`**，直接让异常传播即可。

```java
@Service
public class OrderService {
    @Transactional(rollbackFor = Exception.class)
    public void createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(request));
        try {
            inventoryService.deductStock(request.getProductId(), request.getQuantity());
        } catch (InsufficientStockException e) {
            // 主事务标记回滚
            
            // ✅ 在新事务中保存补偿记录（不受主事务回滚影响）
            orderCompensationService.recordPendingOrder(order.getId(), e.getMessage());
        }
    }
}

@Service
public class OrderCompensationService {
    // REQUIRES_NEW = 挂起当前事务，开启全新独立事务
    @Transactional(propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
    public void recordPendingOrder(Long orderId, String reason) {
        PendingOrder pending = new PendingOrder(orderId, reason);
        pendingOrderRepository.save(pending); // ✅ 这个提交不受主事务回滚影响
    }
}
```