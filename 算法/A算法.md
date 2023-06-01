## 简介
A*算法是一种常用的启发式搜索算法，用于在图形结构中寻找最优路径。它在启发式函数的帮助下，在保证最短路径的前提下，找到了一条最优路径。

A*算法是一种搜索算法，可以用于在图形结构中寻找最短路径。它通过启发式函数来判断哪些节点应该先被搜索。这个启发式函数可以帮助算法选择更有可能是最短路径的节点。

A*算法维护了一个open list和一个closed list。初始时，open list只包含起点，closed list为空。然后，算法重复以下步骤直到找到终点：

1. 从open list中选择一个节点，称为当前节点。
2. 如果当前节点是终点，则算法结束，否则进行下一步。
3. 将当前节点从open list中删除，并将其加入closed list中。
4. 遍历当前节点的所有邻居节点。对于每个邻居节点，进行以下操作：
   - 如果邻居节点已经在closed list中，忽略该节点并继续遍历下一个邻居节点。
   - 如果邻居节点不在open list中，则将该邻居节点加入open list中，并将当前节点作为该邻居节点的前驱节点，同时计算该邻居节点的f、g、h值。其中，g值表示起点到该节点的距离，h值表示该节点到终点的估计距离，f值等于g+h。
   - 如果邻居节点已经在open list中，则计算该节点的新f值。如果该新f值小于该节点之前的f值，则更新该节点的f、g、h值，并将当前节点作为该邻居节点的前驱节点。

A*算法通过启发式函数，选择更有可能是最短路径的节点，可以大大提高搜索的效率。而且，由于A*算法同时使用了Dijkstra算法和贪心算法的优点，因此可以找到一条最短路径。

### 曼哈顿距离
曼哈顿距离，又称为城市街区距离，是指在两点之间，沿着坐标轴的距离总和。比如，点A(1, 2)和点B(6, 4)之间的曼哈顿距离为：|1-6| + |2-4| = 5 + 2 = 7。

下面是一个用C语言实现曼哈顿距离函数的例子：

```c
#include <stdlib.h>

int manhattan_distance(int x1, int y1, int x2, int y2) {
    return abs(x1 - x2) + abs(y1 - y2);
}
```

在函数中，我们使用了C标准库中的`abs`函数，来计算距离坐标轴的距离。`abs`函数可以计算一个整数的绝对值。

下面是一个用C++实现曼哈顿距离函数的例子：

```cpp
#include <cstdlib>

int manhattan_distance(int x1, int y1, int x2, int y2) {
    return abs(x1 - x2) + abs(y1 - y2);
}
```

以上两段代码可以直接用于计算两个点之间的曼哈顿距离，只需要调用这个函数并传入两个点的坐标。

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <cmath>

using namespace std;

// 地图中的一个节点
class Node {
public:
    int x, y;           // 坐标
    double g;           // 从起点到这个节点的代价
    double f;           // 从起点到这个节点的估价代价（g + h）
    Node* parent;       // 父节点，用于记录路径

    Node(int x, int y) {
        this->x = x;
        this->y = y;
        parent = NULL;
        g = f = 0;
    }

    // 计算从当前节点到目标节点的估价代价
    double heuristic_cost(Node* goal) {
        return abs(x - goal->x) + abs(y - goal->y);
    }

    // 比较当前节点的估价代价和另一个节点的估价代价
    bool operator<(const Node& other) const {
        return f > other.f;
    }
};

// 地图
class Map {
public:
    int width, height;
    vector<vector<int>> data;

    Map(int width, int height) {
        this->width = width;
        this->height = height;
        data.resize(width);

        for (int x = 0; x < width; x++) {
            data[x].resize(height);
            for (int y = 0; y < height; y++) {
                data[x][y] = 0;
            }
        }
    }

    // 设置某个坐标的障碍物状态
    void set_obstacle(int x, int y, bool obstacle) {
        data[x][y] = obstacle ? 1 : 0;
    }

    // 判断某个坐标是否为障碍物
    bool is_obstacle(int x, int y) {
        return data[x][y] == 1;
    }

    // 打印地图
    void print() {
        for (int y = 0; y < height; y++) {
            for (int x = 0; x < width; x++) {
                if (is_obstacle(x, y)) {
                    cout << "# ";
                }
                else {
                    cout << ". ";
                }
            }
            cout << endl;
        }
    }
};

// A*算法
vector<Node*> a_star(Map* map, Node* start, Node* goal) {
    vector<Node*> path;
    priority_queue<Node*> openList;
    vector<Node*> closedList;
    openList.push(start);

    while (!openList.empty()) {
        // 取出f值最小的节点
        Node* current = openList.top();
        openList.pop();

        // 如果当前节点是目标节点，就返回路径
        if (current->x == goal->x && current->y == goal->y) {
            while (current != NULL) {
                path.insert(path.begin(), current);
                current = current->parent;
            }
            return path;
        }

        // 扩展当前节点的邻居
        for (int x = -1; x <= 1; x++) {
            for (int y = -1; y <= 1; y++) {
                if (x == 0 && y == 0) {
                    continue;
                }

                int nextX = current->x + x;
                int nextY = current->y + y;
                double nextG = current->g + 1;  // 两个相邻节点代价为1
                double nextF = nextG + Node(nextX, nextY).heuristic_cost(goal);

                // 如果邻居节点已经在closedList中，就跳过
                bool inClosedList = false;
                for (Node* node : closedList) {
                    if (node->x == nextX && node->y == nextY) {
                        inClosedList = true;
                        break;
                    }
                }

                if (inClosedList) {
                    continue;
                }

                // 如果邻居节点已经在openList中，就更新它的代价和父节点
                bool inOpenList = false;
                Node* nextNode = new Node(nextX, nextY);
                nextNode->g = nextG;
                nextNode->f = nextF;
                nextNode->parent = current;

                for (Node* node : openList) {
                    if (node->x == nextX && node->y == nextY && node->f < nextNode->f) {
                        inOpenList = true;
                        break;
                    }
                }

                if (inOpenList) {
                    continue;
                }

                // 将邻居节点加入openList中
                openList.push(nextNode);
            }
        }

        // 将当前节点加入closedList中
        closedList.push_back(current);
    }

    // 如果没有找到路径，就返回一个空的路径
    return path;
}

int main() {
    Map* map = new Map(10, 10);
    map->set_obstacle(1, 3, true);
    map->set_obstacle(2, 3, true);
    map->set_obstacle(3, 3, true);

    Node* start = new Node(1, 1);
    Node* goal = new Node(9, 9);

    vector<Node*> path = a_star(map, start, goal);

    if (!path.empty()) {
        cout << "Path found:" << endl;
        for (Node* node : path) {
            cout << "(" << node->x << ", " << node->y << ")" << endl;
        }
    } else {
        cout << "Path not found." << endl;
    }

    return 0;
}
```