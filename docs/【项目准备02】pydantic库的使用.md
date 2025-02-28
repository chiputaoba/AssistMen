# pydantic的功能
- **数据验证**：使用 Pydantic 可以定义数据模型，明确规定数据应有的字段和类型。当接收到请求数据或准备发送响应数据时，通过 Pydantic 模型进行验证，若数据不符合定义的模型规范，就会抛出错误，提醒开发者数据存在问题。例如在文档中的代码示例，定义了`User`模型，其中`id`是`int`类型，`name`是`str`类型，`email`是`EmailStr`（一种特殊的字符串类型，用于验证是否为有效的邮箱格式）类型，当使用`User.model_validate(response.json())`对从 API 获取的数据进行验证时，若数据不满足这些类型要求，验证就会失败。
- **数据序列化**：将数据转换为适合在网络上传输或存储的格式，比如将 Python 中的数据对象转换为 JSON 格式。Pydantic 模型可以方便地将符合模型定义的数据转换为 JSON 格式，用于响应客户端请求；也能在接收到 JSON 数据时，将其转换为符合模型定义的 Python 数据对象，便于在程序中进一步处理。

# 哪些类和方法可以用在大模型agent开发中，怎么用在大模型agent开发中
1. `BaseModel`类：用于定义数据模型，通过继承它并定义字段作为注解属性，来规范数据的结构和类型。在大模型 agent 处理输入数据、输出结果或中间状态数据时，可利用`BaseModel`定义相应的数据模型，确保数据符合预期格式。在定义用户输入数据模型时：
   ```python
   from pydantic import BaseModel
   class UserInput(BaseModel):
       query: str
       num_results: int = 5
   ```
