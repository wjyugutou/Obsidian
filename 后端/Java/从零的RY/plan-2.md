## 初识 lombok

- @Data
	- 自动生成 getter/setter

## Jackson

序列化 json

- @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
- @JsonInclude(JsonInclude.Include.NON_EMPTY)

```java

@Data  
public class BaseEntity {  
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")  
    private LocalDateTime createTime;  
  
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")  
    private LocalDateTime updateTime;  
  
    private int createBy;  
  
    private int updateBy;  
  
    // 方便获取查询参数  
    @JsonInclude(JsonInclude.Include.NON_EMPTY)  
    private HashMap<String,Object> params;  
  
    public Map<String,Object> getParams() {  
        if(params == null)  
            params = new HashMap<>();  
        return params;  
    }  
}

```