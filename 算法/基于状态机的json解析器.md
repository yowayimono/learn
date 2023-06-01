#### 基于有限状态机的json解析器
JSON结构分析，JSON有以下基础类型：
- object
- string
- array
- number
- bool
- null

是由一系列键值对组成，键值对由冒号隔开，值可以是六种基础类型任意一种，值是字符串，所有一颗套娃，各种套娃但是整个json是一个对象的形式。大概长这样：
```json
{
        "age": 18,
        "isStudent": true,
        "name": "Tom",
        "scores": [
            80,
            90,
            95
        ]
}
```
`[]`这就是数组，`{}`,就是对象，在这里我们肯定要先定义类型`JsonValue`,我们用一个枚举类来表示类型，
```cpp
enum class JsonValueType {
    Null,
    Boolean,
    Number,
    String,
    Array,
    Object
};
```
用一个类来表示 一个JSON，当然这个类要可以表示一个完整的json，我们让他有一个`JsonValueType`,表示他当前是什么类型，然后取这个类型的值就可以
```cpp
private:
    JsonValueType type_;
    bool bool_;
    double number_;
    std::string string_;
    std::vector<JsonValue> array_;
    std::map<std::string, JsonValue> object_;
```
就是这样，当然，我们可以用一个`union`，大大压缩空间，一般也是这样做，当然想在简历上吹嘘一番就用新特性`std::variant`,更安全更强大的代替union的新特性。当时我们这里用第一种方法。给出这个类：
```cpp
class JsonValue {
public:
    JsonValue() : type_(JsonValueType::Null) {}
    JsonValue(bool b) : type_(JsonValueType::Boolean), bool_(b) {}
    JsonValue(double d) : type_(JsonValueType::Number), number_(d) {}
    JsonValue(const std::string& s) : type_(JsonValueType::String), string_(s) {}
    JsonValue(const std::vector<JsonValue>& v) : type_(JsonValueType::Array), array_(v) {}
    JsonValue(const std::map<std::string, JsonValue>& m) : type_(JsonValueType::Object), object_(m) {}

    JsonValueType type() const { return type_; }

    bool asBool() const {
        if (type_ == JsonValueType::Boolean) {
            return bool_;
        }
        return false;
    }

    double asNumber() const {
        if (type_ == JsonValueType::Number) {
            return number_;
        }
        return 0.0;
    }

    std::string asString() const {
        if (type_ == JsonValueType::String) {
            return string_;
        }
        return "";
    }

    std::vector<JsonValue> asArray() const {
        if (type_ == JsonValueType::Array) {
            return array_;
        }
        return std::vector<JsonValue>();
    }

    std::map<std::string, JsonValue> asObject() const {
        if (type_ == JsonValueType::Object) {
            return object_;
        }
        return std::map<std::string, JsonValue>();
    }

    std::string toJsonString(int indent = 0) const {
        std::string result;
        switch (type_) {
            case JsonValueType::Null:
                result = "null";
                break;
            case JsonValueType::Boolean:
                result = bool_ ? "true" : "false";
                break;
            case JsonValueType::Number:
                result = std::to_string(number_);
                break;
            case JsonValueType::String:
                result = "\"" + string_ + "\"";
                break;
            case JsonValueType::Array:
                result = "[\n";
                for (size_t i = 0; i < array_.size(); ++i) {
                    result += std::string(indent + 2, ' ') + array_[i].toJsonString(indent + 2);
                    if (i != array_.size() - 1) {
                        result += ",";
                    }
                    result += "\n";
                }
                result += std::string(indent, ' ') + "]";
                break;
            case JsonValueType::Object:
                result = "{\n";
                for (auto it = object_.begin(); it != object_.end(); ++it) {
                    result += std::string(indent + 2, ' ') + "\"" + it->first + "\": " + it->second.toJsonString(indent + 2);
                    if (it != --object_.end()) {
                        result += ",";
                    }
                    result += "\n";
                }
                result += std::string(indent, ' ') + "}";
                break;
        }
        return result;
    }

private:
    JsonValueType type_;
    bool bool_;
    double number_;
    std::string string_;
    std::vector<JsonValue> array_;
    std::map<std::string, JsonValue> object_;
};
```
`JsonValue`类中的方法：

- `JsonValue()`: 默认构造函数，将type_成员初始化为JsonValueType::Null。


- `JsonValue(bool b)`: 构造函数，将type_成员初始化为`JsonValueType::Boolean`，并将布尔值存储在bool_成员中。



- `JsonValue(double d)`: 构造函数，将type_成员初始化为`JsonValueType::Number`，并将数值存储在`number_`成员中。


- `JsonValue(const std::string& s)`: 构造函数，将`type_`成员初始化为`JsonValueType::String`，并将字符串值存储在`string_`成员中。


- JsonValue(const std::vector<JsonValue>& v): 构造函数，将type_成员初始化为JsonValueType::Array，并将JsonValue对象的向量存储在array_成员中。



- `JsonValue(const std::map<std::string, JsonValue>& m)`: 构造函数，将type_成员初始化为`JsonValueType::Object`，并将字符串键和JsonValue值的映射存储在`object_`成员中。

- `JsonValueType type() const`: 返回JsonValue对象的类型。

- `bool asBool() const`: 如果JsonValue对象的类型为JsonValueType::Boolean，则返回存储的布尔值。否则，返回false。

- `double asNumber() const`: 如果JsonValue对象的类型为`JsonValueType::Number`，则返回存储的数值。否则，返回0.0。

- std::string asString() const: 如果JsonValue对象的类型为JsonValueType::String，则返回存储的字符串值。否则，返回空字符串。

- `std::vector<JsonValue> asArray() const:` 如果`JsonValue`对象的类型为`JsonValueType::Array`，则返回存储的`JsonValue`对象的向量。否则，返回空向量。


- `std::map<std::string, JsonValue> asObject() const`: 如果`JsonValue`对象的类型为`JsonValueType::Object`，则返回存储的字符串键和`JsonValue`值的映射。否则，返回空映射。

- `std::string toJsonString(int indent = 0) const`: 将JsonValue对象转换为对应的JSON字符串表示。`indent`参数指定结果JSON字符串的缩进空格数。

`JsonParser`类:

- `JsonParser(const std::string& json)`: 构造函数，接受一个JSON字符串作为输入，并初始化json_和pos_成员。

- `JsonValue parse()`: 解析存储在json_成员中的JSON字符串，并返回相应的JsonValue对象。此方法充当解析的入口点。

- `void skipWhiteSpace()`: 跳过JSON字符串中的空白字符，通过递增pos_成员直到遇到非空白字符。

- `JsonValue parseObject()`: 解析JSON对象，包括解析对象的键和值，并返回对应的JsonValue对象。

- `JsonValue parseArray()`: 解析JSON数组，包括解析数组的元素，并返回对应的JsonValue对象。

- `JsonValue parseString()`: 解析JSON字符串，并返回对应的JsonValue对象。

- `JsonValue parseBoolean()`: 解析JSON布尔值("true"或"false")，并返回对应的JsonValue对象。

- `JsonValue parseNull()`: 解析JSON空值(null)，并返回对应的JsonValue对象。

- `JsonValue parseNumber()`: 解析JSON数值，并返回对应的JsonValue对象。

- 这些方法共同工作，解析JSON字符串并生成相应的`JsonValue`对象。`JsonParser`类递归调用自己的解析方法来处理嵌套的对象和数组，而`JsonValue`类提供了访问解析值并将其转换回JSON字符串的方法。

```cpp
class JsonParser {
public:
    JsonParser(const std::string& json) : json_(json), pos_(0) {}

    JsonValue parse() {
        skipWhiteSpace();
        char c = json_[pos_];
        if (c == '{') {
            return parseObject();
        } else if (c == '[') {
            return parseArray();
        } else if (c == '\"') {
            return parseString();
        } else if (c == 't' || c == 'f') {
            return parseBoolean();
        } else if (c == 'n') {
            return parseNull();
        } else {
            return parseNumber();
        }
    }

private:
    std::string json_;
    size_t pos_;

    void skipWhiteSpace() {
        while (pos_ < json_.size() && std::isspace(json_[pos_])) {
            ++pos_;
        }
    }

    JsonValue parseObject() {
        std::map<std::string, JsonValue> result;
        ++pos_;
        skipWhiteSpace();
        while (json_[pos_] != '}') {
            std::string key = parseString().asString();
            skipWhiteSpace();
            ++pos_;
            JsonValue value = parse();
            result[key] = value;
            skipWhiteSpace();
            if (json_[pos_] == ',') {
                ++pos_;
                skipWhiteSpace();
            }
        }
        ++pos_;
        return JsonValue(result);
    }

    JsonValue parseArray() {
        std::vector<JsonValue> result;
        ++pos_;
        skipWhiteSpace();
        while (json_[pos_] != ']') {
            JsonValue value = parse();
            result.push_back(value);
            skipWhiteSpace();
            if (json_[pos_] == ',') {
                ++pos_;
                skipWhiteSpace();
            }
        }
        ++pos_;
        return JsonValue(result);
    }

    JsonValue parseString() {
        std::string result;
        ++pos_;
        while (json_[pos_] != '\"') {
            if (json_[pos_] == '\\') {
                ++pos_;
                if (json_[pos_] == '\"') {
                    result += '\"';
                } else if (json_[pos_] == '\\') {
                    result += '\\';
                } else if (json_[pos_] == '/') {
                    result += '/';
                } else if (json_[pos_] == 'b') {
                    result += '\b';
                } else if (json_[pos_] == 'f') {
                    result += '\f';
                } else if (json_[pos_] == 'n') {
                    result += '\n';
                } else if (json_[pos_] == 'r') {
                    result += '\r';
                } else if (json_[pos_] == 't') {
                    result += '\t';
                } else if (json_[pos_] == 'u') {
                    // TODO: handle unicode escape
                }
            } else {
                result += json_[pos_];
            }
            ++pos_;
        }
        ++pos_;
        return JsonValue(result);
    }

    JsonValue parseBoolean() {
        if (json_[pos_] == 't') {
            pos_ += 4;
            return JsonValue(true);
        } else {
            pos_ += 5;
            return JsonValue(false);
        }
    }

    JsonValue parseNull() {
        pos_ += 4;
        return JsonValue();
    }

    JsonValue parseNumber() {
        size_t start = pos_;
        while (pos_ < json_.size() && (std::isdigit(json_[pos_]) || json_[pos_] == '.' || json_[pos_] == 'e' || json_[pos_] == 'E' || json_[pos_] == '+' || json_[pos_] == '-')) {
            ++pos_;
        }
        std::string numberStr = json_.substr(start, pos_ - start);
        double number = std::stod(numberStr);
        return JsonValue(number);
    }
};

```

整体的结构就是一个简单的状态机，有六种状态，但是要注意的是嵌套数组和对象，要进行递归解析，还有各种转义字符处理。

```cpp

int main() {
    std::string json = "{\"name\": \"Alice\", \"age\": 20, \"isStudent\": true, \"grades\": [80, 90, 95], \"address\": {\"city\": \"Beijing\", \"country\": \"China\"}}";
    JsonParser parser(json);
    JsonValue value = parser.parse();
    std::cout << value.toJsonString(2) << std::endl;

    std::map<std::string, JsonValue> object = value.asObject();
    std::cout << "name: " << object["name"].asString() << std::endl;
    std::cout << "age: " << object["age"].asNumber() << std::endl;
    std::cout << "isStudent: " << object["isStudent"].asBool() << std::endl;

    std::vector<JsonValue> grades = object["grades"].asArray();
    std::cout << "grades: [";
    for (size_t i = 0; i < grades.size(); ++i) {
        std::cout << grades[i].asNumber();
        if (i != grades.size() - 1) {
            std::cout << ", ";
        }
    }
    std::cout << "]" << std::endl;

    std::map<std::string, JsonValue> address = object["address"].asObject();
    std::cout << "address: {city: " << address["city"].asString() << ", country: " << address["country"].asString() << "}" << std::endl;

    return 0;
}
```
运行结果:
```json

{
    "address": {
      "city": "Beijing",
      "country": "China"
    },
    "age": 20.000000,
    "grades": [
      80.000000,
      90.000000,
      95.000000
    ],
    "isStudent": true,
    "name": "Alice"
  }
name: Alice
age: 20
isStudent: 1
grades: [80, 90, 95]
address: {city: Beijing, country: China}

```

可能有些同学会对字符串中的斜杆感到疑惑，这是因为字符串中用`\"`表示",转义处理，相对应还有\\这种，`\'`。这就是一个简单的解析器。



```cpp
#include <iostream>
#include <string>
#include <vector>
#include <map>

using namespace std;

enum class JsonValueType {
    Null,
    Boolean,
    Number,
    String,
    Array,
    Object
};

class JsonValue {
public:
    JsonValue() : type(JsonValueType::Null) {}
    JsonValue(bool b) : type(JsonValueType::Boolean), boolValue(b) {}
    JsonValue(double d) : type(JsonValueType::Number), numberValue(d) {}
    JsonValue(const string& s) : type(JsonValueType::String), stringValue(s) {}
    JsonValue(const vector<JsonValue>& v) : type(JsonValueType::Array), arrayValue(v) {}
    JsonValue(const map<string, JsonValue>& m) : type(JsonValueType::Object), objectValue(m) {}

    JsonValueType getType() const { return type; }

    bool asBool() const {
        if (type == JsonValueType::Boolean) {
            return boolValue;
        }
        return false;
    }

    double asNumber() const {
        if (type == JsonValueType::Number) {
            return numberValue;
        }
        return 0.0;
    }

    string asString() const {
        if (type == JsonValueType::String) {
            return stringValue;
        }
        return "";
    }

    vector<JsonValue> asArray() const {
        if (type == JsonValueType::Array) {
            return arrayValue;
        }
        return vector<JsonValue>();
    }

    map<string, JsonValue> asObject() const {
        if (type == JsonValueType::Object) {
            return objectValue;
        }
        return map<string, JsonValue>();
    }

    string toJsonString() const {
        switch (type) {
        case JsonValueType::Null:
            return "null";
        case JsonValueType::Boolean:
            return boolValue ? "true" : "false";
        case JsonValueType::Number:
            return to_string(numberValue);
        case JsonValueType::String:
            return "\"" + stringValue + "\"";
        case JsonValueType::Array: {
            string result = "[";
            for (size_t i = 0; i < arrayValue.size(); ++i) {
                if (i > 0) {
                    result += ",";
                }
                result += arrayValue[i].toJsonString();
            }
            result += "]";
            return result;
        }
        case JsonValueType::Object: {
            string result = "{";
            size_t i = 0;
            for (const auto& kv : objectValue) {
                if (i > 0) {
                    result += ",";
                }
                result += "\"" + kv.first + "\":" + kv.second.toJsonString();
                ++i;
            }
            result += "}";
            return result;
        }
        }
        return "";
    }

private:
    JsonValueType type;
    bool boolValue;
    double numberValue;
    string stringValue;
    vector<JsonValue> arrayValue;
    map<string, JsonValue> objectValue;
};

class JsonParser {
public:
    JsonParser() : pos(0) {}

    JsonValue parse(const string& json) {
        pos = 0;
        return parseValue(json);
    }

private:
    JsonValue parseValue(const string& json) {
        skipWhiteSpace(json);
        char c = json[pos];
        if (c == 'n') {
            parseLiteral(json, "null");
            return JsonValue();
        }
        else if (c == 't') {
            parseLiteral(json, "true");
            return JsonValue(true);
        }
        else if (c == 'f') {
            parseLiteral(json, "false");
            return JsonValue(false);
        }
        else if (c == '\"') {
            return parseString(json);
        }
        else if (c == '[') {
            return parseArray(json);
        }
        else if (c == '{') {
            return parseObject(json);
        }
        else {
            return parseNumber(json);
        }
    }

    void skipWhiteSpace(const string& json) {
        while (pos < json.size() && isspace(json[pos])) {
            ++pos;
        }
    }

    void parseLiteral(const string& json, const string& literal) {
        for (size_t i = 0; i < literal.size(); ++i) {
            if (json[pos + i] != literal[i]) {
                throw runtime_error("Invalid JSON format");
            }
        }
        pos += literal.size();
    }

    JsonValue parseString(const string& json) {
        ++pos;
        string result;
        while (pos < json.size()) {
            char c = json[pos];
            if (c == '\"') {
                ++pos;
                return JsonValue(result);
            }
            else if (c == '\\') {
                ++pos;
                if (pos >= json.size()) {
                    throw runtime_error("Invalid JSON format");
                }
                char escapeChar = json[pos];
                switch (escapeChar) {
                case '\"':
                    result += '\"';
                    break;
                case '\\':
                    result += '\\';
                    break;
                case '/':
                    result += '/';
                    break;
                case 'b':
                    result += '\b';
                    break;
                case 'f':
                    result += '\f';
                    break;
                case 'n':
                    result += '\n';
                    break;
                case 'r':
                    result += '\r';
                    break;
                case 't':
                    result += '\t';
                    break;
                default:
                    throw runtime_error("Invalid JSON format");
                }
            }
            else {
                result += c;
            }
            ++pos;
        }
        throw runtime_error("Invalid JSON format");
    }

    JsonValue parseArray(const string& json) {
        ++pos;
        vector<JsonValue> result;
        while (pos < json.size()) {
            skipWhiteSpace(json);
            if (json[pos] == ']') {
                ++pos;
                return JsonValue(result);
            }
            result.push_back(parseValue(json));
            skipWhiteSpace(json);
            if (json[pos] == ',') {
                ++pos;
            }
            else if (json[pos] == ']') {
                ++pos;
                return JsonValue(result);
            }
            else {
                throw runtime_error("Invalid JSON format");
            }
        }
        throw runtime_error("Invalid JSON format");
    }

    JsonValue parseObject(const string& json) {
        ++pos;
        map<string, JsonValue> result;
        while (pos < json.size()) {
            skipWhiteSpace(json);
            if (json[pos] == '}') {
                ++pos;
                return JsonValue(result);
            }
            string key = parseString(json).asString();
            skipWhiteSpace(json);
            if (json[pos] != ':') {
                throw runtime_error("Invalid JSON format");
            }
            ++pos;
            JsonValue value = parseValue(json);
            result[key] = value;
            skipWhiteSpace(json);
            if (json[pos] == ',') {
                ++pos;
            }
            else if (json[pos] == '}') {
                ++pos;
                return JsonValue(result);
            }
            else {
                throw runtime_error("Invalid JSON format");
            }
        }
        throw runtime_error("Invalid JSON format");
    }

    JsonValue parseNumber(const string& json) {
        size_t start = pos;
        if (json[pos] == '-') {
            ++pos;
        }
        while (pos < json.size() && isdigit(json[pos])) {
            ++pos;
        }
        if (pos < json.size() && json[pos] == '.') {
            ++pos;
            while (pos < json.size() && isdigit(json[pos])) {
                ++pos;
            }
        }
        if (pos < json.size() && (json[pos] == 'e' || json[pos] == 'E')) {
            ++pos;
            if (pos < json.size() && (json[pos] == '+' || json[pos] == '-')) {
                ++pos;
            }
            while (pos < json.size() && isdigit(json[pos])) {
                ++pos;
            }
        }
        double value = stod(json.substr(start, pos - start));
        return JsonValue(value);
    }

    size_t pos;
};

int main() {
    string json = "{\"name\":\"Alice\",\"age\":20,\"isStudent\":true,\"scores\":[80,90,95],\"address\":{\"city\":\"Beijing\",\"country\":\"China\"}}";
    JsonParser parser;
    JsonValue value = parser.parse(json);
    cout << value.toJsonString() << endl;
    return 0;
}


```