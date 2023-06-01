<!--
 * @Author: yowayimono
 * @Date: 2023-05-12 21:24:41
 * @LastEditors: yowayimono
 * @LastEditTime: 2023-06-01 19:35:23
 * @Description: nothing
-->
以下是一个简单的用 C++ 实现 Trie 树的示例代码：

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <algorithm>
using namespace std;

class TrieNode {
public:
    unordered_map<char, TrieNode*> children;
    int count;

    TrieNode() {
        count = 0;
    }

    ~TrieNode() {
        for (auto& child : children) {
            delete child.second;
        }
    }
};

class Trie {
public:
    Trie() {
        root = new TrieNode();
    }

    ~Trie() {
        delete root;
    }

    void insert(string word) {
        TrieNode* node = root;
        for (char c : word) {
            if (node->children.find(c) == node->children.end()) {
                node->children[c] = new TrieNode();
            }
            node = node->children[c];
        }
        node->count++;
    }

    vector<string> search(string prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            if (node->children.find(c) == node->children.end()) {
                return vector<string>();
            }
            node = node->children[c];
        }
        vector<pair<string, int>> matches;
        traverse(node, prefix, matches);
        sort(matches.begin(), matches.end(), [](const pair<string, int>& a, const pair<string, int>& b) {
            return a.second > b.second;
        });
        vector<string> result;
        for (auto& match : matches) {
            result.push_back(match.first);
        }
        return result;
    }

private:
    TrieNode* root;

    void traverse(TrieNode* node, string prefix, vector<pair<string, int>>& matches) {
        if (node->count > 0) {
            matches.push_back(make_pair(prefix, node->count));
        }
        for (auto& child : node->children) {
            traverse(child.second, prefix + child.first, matches);
        }
    }
};

int main() {
    Trie trie;
    trie.insert("abc");
    trie.insert("abcd");
    trie.insert("abd");
    trie.insert("acd");
    trie.insert("bcd");

    vector<string> result = trie.search("a");
    for (string& s : result) {
        cout << s << endl;
    }

    return 0;
}
```
该示例代码中，使用了一个 TrieNode 类来表示 Trie 树的节点，其中 children 成员是一个 unordered_map 类型，用于存储子节点；count 成员是一个计数器，用于记录以该节点为结尾的字符串数量。

使用 Trie 类来封装 Trie 树，实现了 insert 和 search 两个方法，其中 insert 方法用于插入字符串，search 方法用于查找以给定前缀开头的字符串，并返回匹配结果。search 方法中，首先在 Trie 树上匹配输入前缀，然后对以该节点为结尾的所有字符串进行遍历，得到所有可能的匹配结果。最后，按照计数器从大到小排序，以便返回最精确的查询结果。