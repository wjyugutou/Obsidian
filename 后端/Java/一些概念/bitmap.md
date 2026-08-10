使用 ```RoaringBitmap``` 库 ```

```java
RoaringBitmap rb = new RoaringBitmap();
rb.add(1);
rb.add(1_000_000);
rb.contains(1); // true
```
