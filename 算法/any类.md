```cpp
#include <iostream>
#include <typeindex>
#include <typeinfo>
#include <utility>

class any {
public:
    template <typename T>
    any(T value) : m_data(new holder<T>(std::forward<T>(value))) {}

    ~any() {
        delete m_data;
    }

    template <typename T>
    T& get() const {
        return static_cast<holder<T>*>(m_data)->value;
    }

    bool empty() const {
        return m_data == nullptr;
    }

    void reset() {
        delete m_data;
        m_data = nullptr;
    }

    const std::type_info& type() const {
        return m_data ? m_data->type() : typeid(void);
    }

private:
    struct holder_base {
        virtual ~holder_base() {}
        virtual const std::type_info& type() const = 0;
    };

    template <typename T>
    struct holder : holder_base {
        holder(T value) : value(std::move(value)) {}
        const std::type_info& type() const override { return typeid(T); }
        T value;
    };

    holder_base* m_data;
};

std::ostream& operator<<(std::ostream& os, const any& a) {
    if (a.empty()) {
        os << "Empty";
    } else {
        
        if (a.type() == typeid(int)) {
            os << a.get<int>();
        } else if (a.type() == typeid(std::string)) {
            os << a.get<std::string>();
        } else if (a.type() == typeid(double)) {
            os << a.get<double>();
        }
    }

    return os;
}

int main() {
    any a1 = 42;
    any a2 = std::string("Hello, world!");
    any a3 = 3.14;

    std::cout << a1 << std::endl;
    std::cout << a2 << std::endl;
    std::cout << a3 << std::endl;

    a1.reset();
    a2.reset();
    a3.reset();

    std::cout << a1 << std::endl;
    std::cout << a2 << std::endl;
    std::cout << a3 << std::endl;

    return 0;
}
```



```cpp
#include <iostream>
#include <typeindex>
#include <typeinfo>
#include <utility>

class any {
public:
    template <typename T>
    any(T value) : m_data(new holder<T>(std::forward<T>(value))) {}

    any(const any& other) : m_data(other.m_data ? other.m_data->clone() : nullptr) {}

    ~any() {
        delete m_data;
    }

    any& operator=(const any& other) {
        if (this != &other) {
            delete m_data;
            m_data = other.m_data ? other.m_data->clone() : nullptr;
        }
        return *this;
    }

    template <typename T>
    T& get() const {
        return static_cast<holder<T>*>(m_data)->value;
    }

    bool empty() const {
        return m_data == nullptr;
    }

    void reset() {
        delete m_data;
        m_data = nullptr;
    }

    const std::type_info& type() const {
        return m_data ? m_data->type() : typeid(void);
    }

private:
    struct holder_base {
        virtual ~holder_base() {}
        virtual const std::type_info& type() const = 0;
        virtual holder_base* clone() const = 0;
    };

    template <typename T>
    struct holder : holder_base {
        holder(T value) : value(std::move(value)) {}
        const std::type_info& type() const override { return typeid(T); }
        holder_base* clone() const override { return new holder<T>(value); }
        T value;
    };

    holder_base* m_data;
};

std::ostream& operator<<(std::ostream& os, const any& a) {
    if (a.empty()) {
        os << "Empty";
    } else {
        if (a.type() == typeid(int)) {
            os << a.get<int>();
        } else if (a.type() == typeid(std::string)) {
            os << a.get<std::string>();
        } else if (a.type() == typeid(double)) {
            os << a.get<double>();
        } else if (a.type() == typeid(char)) {
            os << a.get<char>();
        } else if (a.type() == typeid(const char*)) {
            os << a.get<const char*>();
        } else if (a.type() == typeid(bool)) {
            os << std::boolalpha << a.get<bool>();
        }
    }

    return os;
}

int main() {
    any a1 = 42;
    any a2 = std::string("Hello, world!");
    any a3 = 3.14;
    any a4 = 'A';
    any a5 = "Hello, any!";
    any a6 = true;

    std::cout << a1 << std::endl;
    std::cout << a2 << std::endl;
    std::cout << a3 << std::endl;
    std::cout << a4 << std::endl;
    std::cout << a5 << std::endl;
    std::cout << a6 << std::endl;

    a1.reset();
    a2.reset();
    a3.reset();
    a4.reset();
    a5.reset();
    a6.reset();

    a1 = 520;
    a2 = 222.222;
    a3 = 'B';
    a4 = "Hello again!";
    a5 = false;

    std::cout << a1 << std::endl;
    std::cout << a2 << std::endl;
    std::cout << a3 << std::endl;
    std::cout << a4 << std::endl;
    std::cout << a5 << std::endl;
    std::cout << a6 << std::endl;

    return 0;
}
```