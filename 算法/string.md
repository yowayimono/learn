```cpp

#include <iostream>
#include <cstring>

class String {
public:
    String() : m_data(nullptr), m_size(0), m_capacity(0) {}
    String(const char* str) : m_data(nullptr), m_size(0), m_capacity(0) {
        append(str);
    }
    String(const String& other) : m_data(nullptr), m_size(0), m_capacity(0) {
        append(other.m_data);
    }
    ~String() {
        if (m_data) {
            delete[] m_data;
        }
    }

    String& operator=(const char* str) {
        clear();
        append(str);
        return *this;
    }
    String& operator=(const String& other) {
        clear();
        append(other.m_data);
        return *this;
    }

    char& operator[](int index) {
        return m_data[index];
    }
    const char& operator[](int index) const {
        return m_data[index];
    }

    int size() const {
        return m_size;
    }
    int capacity() const {
        return m_capacity;
    }
    bool empty() const {
        return m_size == 0;
    }

    void clear() {
        if (m_data) {
            delete[] m_data;
            m_data = nullptr;
        }
        m_size = 0;
        m_capacity = 0;
    }

    void append(const char* str) {
        int len = strlen(str);
        reserve(m_size + len);
        memcpy(m_data + m_size, str, len);
        m_size += len;
        m_data[m_size] = '\0';
    }

    void reserve(int newCapacity) {
        if (newCapacity <= m_capacity) {
            return;
        }
        char* newData = new char[newCapacity + 1];
        if (m_data) {
            memcpy(newData, m_data, m_size);
            delete[] m_data;
        }
        m_data = newData;
        m_capacity = newCapacity;
    }

    char* c_str() const {
        return m_data;
    }

    int find(const char* str, int pos = 0) const {
        int len = strlen(str);
        for (int i = pos; i <= m_size - len; i++) {
            if (memcmp(m_data + i, str, len) == 0) {
                return i;
            }
        }
        return -1;
    }

    String substr(int pos, int len = -1) const {
        if (len == -1) {
            len = m_size - pos;
        }
        String result;
        result.reserve(len);
        memcpy(result.m_data, m_data + pos, len);
        result.m_size = len;
        result.m_data[len] = '\0';
        return result;
    }

    class iterator {
    public:
        iterator(char* ptr) : m_ptr(ptr) {}
        char& operator*() {
            return *m_ptr;
        }
        iterator& operator++() {
            ++m_ptr;
            return *this;
        }
        iterator operator++(int) {
            iterator tmp = *this;
            ++m_ptr;
            return tmp;
        }
        bool operator==(const iterator& other) const {
            return m_ptr == other.m_ptr;
        }
        bool operator!=(const iterator& other) const {
            return m_ptr != other.m_ptr;
        }
    private:
        char* m_ptr;
    };

    class const_iterator {
    public:
        const_iterator(const char* ptr) : m_ptr(ptr) {}
        const char& operator*() const {
            return *m_ptr;
        }
        const_iterator& operator++() {
            ++m_ptr;
            return *this;
        }
        const_iterator operator++(int) {
            const_iterator tmp = *this;
            ++m_ptr;
            return tmp;
        }
        bool operator==(const const_iterator& other) const {
            return m_ptr == other.m_ptr;
        }
        bool operator!=(const const_iterator& other) const {
            return m_ptr != other.m_ptr;
        }
    private:
        const char* m_ptr;
    };

    iterator begin() {
        return iterator(m_data);
    }
    iterator end() {
        return iterator(m_data + m_size);
    }
    const_iterator begin() const {
        return const_iterator(m_data);
    }
    const_iterator end() const s
        return const_iterator(m_data + m_size);
    }

private:
    char* m_data;
    int m_size;
    int m_capacity;
};

int main() {
    String s1("hello");
    String s2(s1);
    String s3 = "world";
    s1.append("!");
    s2 = s1;
    std::cout << s1.c_str() << std::endl;
    std::cout << s2.c_str() << std::endl;
    std::cout << s3.c_str() << std::endl;
    std::cout << s1.find("lo") << std::endl;
    std::cout << s1.substr(1, 3).c_str() << std::endl;
    for (auto it = s1.begin(); it != s1.end(); ++it) {
        std::cout << *it;
    }
    std::cout << std::endl;
    for (auto c : s1) {
        std::cout << c;
    }
    std::cout << std::endl;
    return 0;
}
```

这个实现提供了以下接口和构造函数：

- String()：默认构造函数，创建一个空字符串。
- String(const char* str)：构造函数，创建一个包含指定 C 字符串的字符串。
 - String(const String& other)：拷贝构造函数，创建一个包含另一个字符串的副本。
- ~String()：析构函数，释放字符串占用的内存。
- operator=(const char* str)：赋值运算符，将字符串设置为指定 C 字符串。
- operator=(const String& other)：赋值运算符，将字符串设置为另一个字符串的副本。
- operator[](int index)：下标运算符，返回指定位置的字符的引用。
- operator[](int index) const：下标运算符，返回指定位置的字符的常引用。
- size()：返回字符串的长度。
- capacity()：返回字符串的容量。
- empty()：返回字符串是否为空。
- clear()：清空字符串。
- append(const char* str)：将指定 C 字符串追加到字符串末尾。
- reserve(int newCapacity)：将字符串的容量扩展到指定大小。
- c_str()：返回字符串的 C 字符串表示。
- find(const char* str, int pos = 0)：在字符串中查找指定 C 字符串的位置，从指定位置开始查找。
- substr(int pos, int len = -1)：返回从指定位置开始的指定长度的子字符串。
- begin()：返回一个迭代器，指向字符串的第一个字符。
- end()：返回一个迭代器，指向字符串的最后一个字符的下一个位置。
- 此外，还提供了两个迭代器类 iterator 和 const_iterator，用于访问字符串的字符。这两个类都实现了以下接口：

- operator*()：返回迭代器指向的字符的引用。
- operator++()：将迭代器向前移动一个位置，并返回自身的引用。
- operator++(int)：将迭代器向前移动一个位置，并返回移动前的副本。
- operator==()：比较两个迭代器是否相等。
- operator!=()：比较两个迭代器是否不相等。
最后，还演示了如何使用迭代器遍历字符串。




v_0.2
```cpp
#include <iostream>
#include <cstring>

class String {
public:
    // 默认构造函数
    String() : data_(nullptr), length_(0) {}

    // 构造函数：接受C风格字符串
    String(const char* str) {
        length_ = std::strlen(str);
        data_ = new char[length_ + 1];
        std::strcpy(data_, str);
    }

    // 构造函数：接受指定长度的字符
    String(size_t length, char c) {
        data_ = new char[length + 1];
        std::memset(data_, c, length);
        data_[length] = '\0';
        length_ = length;
    }

    // 拷贝构造函数
    String(const String& other) {
        length_ = other.length_;
        data_ = new char[length_ + 1];
        std::strcpy(data_, other.data_);
    }

    // 移动构造函数
    String(String&& other) noexcept {
        length_ = other.length_;
        data_ = other.data_;
        other.length_ = 0;
        other.data_ = nullptr;
    }

    // 析构函数
    ~String() {
        delete[] data_;
    }

    // 获取字符串长度
    size_t length() const {
        return length_;
    }

    // 判断字符串是否为空
    bool empty() const {
        return length_ == 0;
    }

    // 获取C风格字符串
    const char* c_str() const {
        return data_;
    }

    // 查找子串
    size_t find(const String& substr) const {
        const char* result = std::strstr(data_, substr.data_);
        if (result) {
            return result - data_;
        }
        return std::string::npos;
    }

    // 查询切片
    String slice(size_t start, size_t end) const {
        if (start >= length_ || start > end) {
            return String();
        }
        if (end >= length_) {
            end = length_ - 1;
        }
        size_t slice_length = end - start + 1;
        char* slice_data = new char[slice_length + 1];
        std::strncpy(slice_data, data_ + start, slice_length);
        slice_data[slice_length] = '\0';
        return String(slice_data);
    }

    // 字符串比较
    int compare(const String& other) const {
        return std::strcmp(data_, other.data_);
    }

    // 重载赋值运算符
    String& operator=(const String& other) {
        if (this != &other) {
            delete[] data_;
            length_ = other.length_;
            data_ = new char[length_ + 1];
            std::strcpy(data_, other.data_);
        }
        return *this;
    }

    // 重载加法运算符
    String operator+(const String& other) const {
        size_t new_length = length_ + other.length_;
        char* new_data = new char[new_length + 1];
        std::strcpy(new_data, data_);
        std::strcat(new_data, other.data_);
        return String(new_data);
    }

    // 迭代器支持：开始位置迭代器
    const char* begin() const {
        return data_;
    }

    // 迭代器支持：结束位置迭代器
    const char* end() const {
        return data_ + length_;
    }

private:
    char* data_;
    size_t length_;
};

int main() {
    // 构造函数演示
    String str1; // 默认构造函数
    String str2("Hello"); // 接受C风格字符串的构造函数
    String str3(5, 'a'); // 接受指定长度和字符的构造函数

   for(auto& x:str2){
       std::cout<<x<<' ';
   }

    return 0;
}

```

v_0.3
```cpp
