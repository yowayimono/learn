```cpp
#include <iostream>
#include <cassert>

template <typename T>
class Vector {
public:
    // 构造函数
    Vector() : data(nullptr), size(0), capacity(0) {}

    Vector(int initialSize) : size(initialSize), capacity(initialSize) {
        data = new T[capacity];
    }

    // 析构函数
    ~Vector() {
        delete[] data;
    }

    // 获取向量大小
    int getSize() const {
        return size;
    }

    // 获取向量容量
    int getCapacity() const {
        return capacity;
    }

    // 判断向量是否为空
    bool isEmpty() const {
        return size == 0;
    }

    // 访问元素
    T& operator[](int index) {
        assert(index >= 0 && index < size);
        return data[index];
    }

    const T& operator[](int index) const {
        assert(index >= 0 && index < size);
        return data[index];
    }

    // 添加元素
    void pushBack(const T& value) {
        if (size == capacity) {
            resize(capacity * 2);
        }
        data[size++] = value;
    }

    // 删除最后一个元素
    void popBack() {
        assert(size > 0);
        size--;
    }

    // 清空向量
    void clear() {
        size = 0;
    }

    // 迭代器
    class Iterator {
    public:
        Iterator(T* ptr) : ptr(ptr) {}

        T& operator*() const {
            return *ptr;
        }

        Iterator& operator++() {
            ++ptr;
            return *this;
        }

        bool operator!=(const Iterator& other) const {
            return ptr != other.ptr;
        }

    private:
        T* ptr;
    };

    Iterator begin() {
        return Iterator(data);
    }

    Iterator end() {
        return Iterator(data + size);
    }

private:
    T* data;
    int size;
    int capacity;

    // 调整容量
    void resize(int newCapacity) {
        T* newData = new T[newCapacity];
        for (int i = 0; i < size; ++i) {
            newData[i] = data[i];
        }
        delete[] data;
        data = newData;
        capacity = newCapacity;
    }
};

int main() {
    Vector<int> vec;
    vec.pushBack(1);
    vec.pushBack(2);
    vec.pushBack(3);

    for (int i = 0; i < vec.getSize(); ++i) {
        std::cout << vec[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```


v_0.2


```cpp
#include <iostream>
#include <memory>

template<typename T>
class Vector {
public:
    // 默认构造函数
    Vector() : size_(0), capacity_(0), data_(nullptr) {}

    // 带初始大小的构造函数
    explicit Vector(size_t size) : size_(size), capacity_(size), data_(new T[size]) {}

    // 带初始值和大小的构造函数
    Vector(size_t size, const T& value) : size_(size), capacity_(size), data_(new T[size]) {
        for (size_t i = 0; i < size_; ++i) {
            data_[i] = value;
        }
    }

    // 拷贝构造函数
    Vector(const Vector& other) : size_(other.size_), capacity_(other.size_), data_(new T[other.size_]) {
        for (size_t i = 0; i < size_; ++i) {
            data_[i] = other.data_[i];
        }
    }

    // 移动构造函数
    Vector(Vector&& other) noexcept : size_(other.size_), capacity_(other.capacity_), data_(other.data_) {
        other.size_ = 0;
        other.capacity_ = 0;
        other.data_ = nullptr;
    }

    // 析构函数
    ~Vector() {
        delete[] data_;
    }

    // 拷贝赋值运算符
    Vector& operator=(const Vector& other) {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            capacity_ = other.size_;
            data_ = new T[other.size_];
            for (size_t i = 0; i < size_; ++i) {
                data_[i] = other.data_[i];
            }
        }
        return *this;
    }

    // 移动赋值运算符
    Vector& operator=(Vector&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            size_ = other.size_;
            capacity_ = other.capacity_;
            data_ = other.data_;
            other.size_ = 0;
            other.capacity_ = 0;
            other.data_ = nullptr;
        }
        return *this;
    }

    // 获取当前元素个数
    size_t size() const {
        return size_;
    }

    // 获取当前容量
    size_t capacity() const {
        return capacity_;
    }

    // 判断容器是否为空
    bool empty() const {
        return size_ == 0;
    }

    // 重设容器大小
    void resize(size_t size) {
        if (size > capacity_) {
            reserve(size);
        }
        if (size > size_) {
            for (size_t i = size_; i < size; ++i) {
                data_[i] = T();
            }
        }
        size_ = size;
    }

    // 重设容器大小，并用指定的值填充新元素
    void resize(size_t size, const T& value) {
        if (size > capacity_) {
            reserve(size);
        }
        if (size > size_) {
            for (size_t i = size_; i < size; ++i) {
                data_[i] = value;
            }
        }
        size_ = size;
    }

    // 增加容器的容量
    void reserve(size_t capacity) {
        if (capacity > capacity_) {
            T* newData = new T[capacity];
            for (size_t i = 0; i < size_; ++i) {
                newData[i] = data_[i];
            }
            delete[] data_;
            data_ = newData;
            capacity_ = capacity;
        }
    }

    // 清空容器
    void clear() {
        delete[] data_;
        size_ = 0;
        capacity_ = 0;
        data_ = nullptr;
    }

    // 在末尾添加一个元素
    void push_back(const T& value) {
        if (size_ == capacity_) {
            reserve(capacity_ == 0 ? 1 : capacity_ * 2);
        }
        data_[size_] = value;
        ++size_;
    }

    // 在末尾添加一个元素（移动语义）
    void push_back(T&& value) {
        if (size_ == capacity_) {
            reserve(capacity_ == 0 ? 1 : capacity_ * 2);
        }
        data_[size_] = std::move(value);
        ++size_;
    }

    // 删除末尾的元素
    void pop_back() {
        if (size_ > 0) {
            --size_;
        }
    }
    
     // 在指定位置插入元素
    void insert(size_t index, const T& value) {
        if (index > size_) {
            throw std::out_of_range("Index out of range");
        }

        if (size_ == capacity_) {
            // 扩容
            reserve(capacity_ * 2);
        }

        // 将[index, size_)范围内的元素向后移动一位
        for (size_t i = size_; i > index; --i) {
            data_[i] = data_[i - 1];
        }

        // 在index位置插入新元素
        data_[index] = value;
        ++size_;
    }
    
    
      void erase(size_t index) {
        if (index >= size_) {
            throw std::out_of_range("Index out of range");
        }

        // 将[index + 1, size_)范围内的元素向前移动一位
        for (size_t i = index; i < size_ - 1; ++i) {
            data_[i] = data_[i + 1];
        }

        --size_;
    }
    
    
    

    // 访问容器中的元素
    T& operator[](size_t index) {
        return data_[index];
    }

    const T& operator[](size_t index) const {
        return data_[index];
    }
    
    class Iterator {
    public:
        using iterator_category = std::random_access_iterator_tag;
        using value_type = T;
        using difference_type = std::ptrdiff_t;
        using pointer = T*;
        using reference = T&;

        // 构造函数
        Iterator(pointer ptr) : ptr_(ptr) {}

        // 解引用操作符
        reference operator*() const {
            return *ptr_;
        }

        // 前缀递增操作符
        Iterator& operator++() {
            ++ptr_;
            return *this;
        }

        // 后缀递增操作符
        Iterator operator++(int) {
            Iterator temp = *this;
            ++ptr_;
            return temp;
        }

        // 前缀递减操作符
        Iterator& operator--() {
            --ptr_;
            return *this;
        }

        // 后缀递减操作符
        Iterator operator--(int) {
            Iterator temp = *this;
            --ptr_;
            return temp;
        }

        // 加法操作符
        Iterator operator+(difference_type n) const {
            return Iterator(ptr_ + n);
        }

        // 减法操作符
        Iterator operator-(difference_type n) const {
            return Iterator(ptr_ - n);
        }

        // 复合赋值加法操作符
        Iterator& operator+=(difference_type n) {
            ptr_ += n;
            return *this;
        }

        // 复合赋值减法操作符
        Iterator& operator-=(difference_type n) {
            ptr_ -= n;
            return *this;
        }

        // 下标操作符
        reference operator[](difference_type n) const {
            return *(ptr_ + n);
        }

        // 比较操作符
        bool operator==(const Iterator& other) const {
            return ptr_ == other.ptr_;
        }

        bool operator!=(const Iterator& other) const {
            return ptr_ != other.ptr_;
        }

        bool operator<(const Iterator& other) const {
            return ptr_ < other.ptr_;
        }

        bool operator>(const Iterator& other) const {
            return ptr_ > other.ptr_;
        }

        bool operator<=(const Iterator& other) const {
            return ptr_ <= other.ptr_;
        }

        bool operator>=(const Iterator& other) const {
            return ptr_ >= other.ptr_;
        }

        // 获取指针
        pointer get() const {
            return ptr_;
        }

    private:
        pointer ptr_;
    };

    // 迭代器访问方法
    Iterator begin() {
        return Iterator(data_);
    }

    Iterator end() {
        return Iterator(data_ + size_);
    }
    
     T min() const {
        if (empty()) {
            throw std::runtime_error("Vector is empty");
        }

        T min_value = data_[0];
        for (size_t i = 1; i < size_; ++i) {
            if (data_[i] < min_value) {
                min_value = data_[i];
            }
        }

        return min_value;
    }

    // 获取最大值
    T max() const {
        if (empty()) {
            throw std::runtime_error("Vector is empty");
        }

        T max_value = data_[0];
        for (size_t i = 1; i < size_; ++i) {
            if (data_[i] > max_value) {
                max_value = data_[i];
            }
        }

        return max_value;
    }

    // 排序
    void sort() {
        std::sort(data_, data_ + size_);
    }
private:
    size_t size_;         // 元素个数
    size_t capacity_;     // 容量
    T* data_;             // 数据指针
};


int main(){
    Vector<int> vec(10,520);
    vec.insert(5,514);
    vec.sort();
    for(auto &x:vec){
        std::cout<<x<<' ';
    }
}


```

