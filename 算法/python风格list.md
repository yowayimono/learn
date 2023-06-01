```cpp
#include <iostream>
#include <vector>
#include <variant>
#include <string>

class PythonList {
private:
    std::vector<std::variant<int, std::string>> data;

public:
    void append(const std::variant<int, std::string>& item) {
        data.push_back(item);
    }

    std::string toString() const {
        std::string result = "[";
        bool first = true;
        for (const auto& item : data) {
            if (!first) {
                result += ", ";
            }
            if (std::holds_alternative<int>(item)) {
                result += std::to_string(std::get<int>(item));
            } else if (std::holds_alternative<std::string>(item)) {
                result += "\"" + std::get<std::string>(item) + "\"";
            }
            first = false;
        }
        result += "]";
        return result;
    }

    std::variant<int, std::string>& operator[](size_t index) {
        return data[index];
    }

    const std::variant<int, std::string>& operator[](size_t index) const {
        return data[index];
    }

    // Iterator implementation
    typedef typename std::vector<std::variant<int, std::string>>::iterator iterator;
    typedef typename std::vector<std::variant<int, std::string>>::const_iterator const_iterator;

    iterator begin() {
        return data.begin();
    }

    iterator end() {
        return data.end();
    }

    const_iterator begin() const {
        return data.begin();
    }

    const_iterator end() const {
        return data.end();
    }
};

std::ostream& operator<<(std::ostream& os, const std::variant<int, std::string>& item) {
    if (std::holds_alternative<int>(item)) {
        os << std::get<int>(item);
    } else if (std::holds_alternative<std::string>(item)) {
        os << "\"" << std::get<std::string>(item) << "\"";
    }
    return os;
}

int main() {
    PythonList pyList;
    pyList.append(1);
    pyList.append("sad");
    pyList.append(23);
    pyList.append("Hi");

    std::cout << "PythonList: " << pyList.toString() << std::endl;

    std::cout << "Iterating through the PythonList: ";
    for (const auto& item : pyList) {
        std::cout << item << " ";
    }
    std::cout << std::endl;

    std::cout << "Accessing items using indices:" << std::endl;
    std::cout << "Item at index 0: " << pyList[0] << std::endl;
    std::cout << "Item at index 2: " << pyList[2] << std::endl;

    pyList[1] = "happy";
    std::cout << "PythonList after modifying item at index 1: " << pyList.toString() << std::endl;

    return 0;
}
```




```cpp
#include <iostream>
#include <vector>
#include <variant>
#include <string>

class PythonList {
private:
    std::vector<std::variant<int, std::string, PythonList>> data;

public:
    void append(const std::variant<int, std::string, PythonList>& item) {
        data.push_back(item);
    }

    std::string toString() const {
        std::string result = "[";
        bool first = true;
        for (const auto& item : data) {
            if (!first) {
                result += ", ";
            }
            if (std::holds_alternative<int>(item)) {
                result += std::to_string(std::get<int>(item));
            } else if (std::holds_alternative<std::string>(item)) {
                result += "\"" + std::get<std::string>(item) + "\"";
            } else if (std::holds_alternative<PythonList>(item)) {
                result += std::get<PythonList>(item).toString();
            }
            first = false;
        }
        result += "]";
        return result;
    }

    std::variant<int, std::string, PythonList>& operator[](size_t index) {
        return data[index];
    }

    const std::variant<int, std::string, PythonList>& operator[](size_t index) const {
        return data[index];
    }

    // Iterator implementation
    typedef typename std::vector<std::variant<int, std::string, PythonList>>::iterator iterator;
    typedef typename std::vector<std::variant<int, std::string, PythonList>>::const_iterator const_iterator;

    iterator begin() {
        return data.begin();
    }

    iterator end() {
        return data.end();
    }

    const_iterator begin() const {
        return data.begin();
    }

    const_iterator end() const {
        return data.end();
    }
};

std::ostream& operator<<(std::ostream& os, const std::variant<int, std::string, PythonList>& item) {
    if (std::holds_alternative<int>(item)) {
        os << std::get<int>(item);
    } else if (std::holds_alternative<std::string>(item)) {
        os << "\"" << std::get<std::string>(item) << "\"";
    } else if (std::holds_alternative<PythonList>(item)) {
        os << std::get<PythonList>(item).toString();
    }
    return os;
}

int main() {
    PythonList pyList;
    pyList.append(1);
    pyList.append("sad");
    pyList.append(23);
    pyList.append("Hi");

    PythonList innerList;
    innerList.append("inner");
    innerList.append(42);
    pyList.append(innerList);

    std::cout << "PythonList: " << pyList.toString() << std::endl;

    std::cout << "Iterating through the PythonList: ";
    for (const auto& item : pyList) {
        std::cout << item << " ";
    }
    std::cout << std::endl;

    std::cout << "Accessing items using indices:" << std::endl;
    std::cout << "Item at index 0: " << pyList[0] << std::endl;
    std::cout << "Item at index 2: " << pyList[2] << std::endl;

    pyList[1] = "happy";
    std::cout << "PythonList after modifying an item: " << pyList.toString() << std::endl;
    
    
    std::cout<<'\n'<<'\n'<<'\n';
    PythonList list;
    list.append(innerList);
    list.append(pyList);
    list.append("sbsbsbsbsbsb");
    
    std::cout<<list.toString()<<std::endl;

    return 0;
}
```