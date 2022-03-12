没想好如何描述
```cpp
#include <iostream>
#include <algorithm>
#include <set>
#include <queue>
#include <cstdio>
#include <vector>
#include <map>
#include <stack>
#include <cmath>
#include <string>

using namespace std;
typedef unsigned long long uint64_t;
typedef unsigned int uint32_t;
typedef int int32_t;

class Node {
public:
	// 仿下leveldb的写法。OJ不支持 我不是很认同
	/*explicit*/ Node(const int32_t start_);
//	Node(const Node&) = delete;
//	Node& operator=(const Node&) = delete;
	
	int32_t getMove(char ch);
	void addMap(char ch, int32_t v) { directions_[ch] = v; }
	void setStart(int32_t start_) { start_ = start_; }
	int32_t getStart() { return start_; }
private:
	int32_t start_;
	map<char, int32_t> directions_;
};

Node::Node(const int32_t start_) : start_(start_) {}
int32_t Node::getMove(char ch) {
	return directions_[ch] == 0 ? -1 : directions_[ch];
}

vector<Node*> nodes_(10);
vector<int32_t> final_state, res;
int32_t start_, end_, n;
Node* now_ptr;

int main(int argc, char *argv[]) {
	int32_t tmp = 0, s, e;
	char ch;
	cin >> start_ >> end_;
	for (int32_t i = 0; i < end_; i++) {
		cin >> tmp;
		final_state.push_back(tmp);
	}
	
	cin >> n;
	for (int32_t i = 0; i < n; i++) {
		cin >> s >> ch >> e;
		if (nodes_[s] == nullptr) {
			nodes_[s] = new Node(s);
		}
		nodes_[s]->addMap(ch, e);
	}
	
	now_ptr = nodes_[start_];
	while(true) {
		ch = getchar();
		if (ch == '\n') {
			continue;
		}
		if (ch == EOF) {
			break;
		}

		if ((tmp = now_ptr->getMove(ch)) > 0) {
			res.push_back(tmp);
			now_ptr = nodes_[tmp];
		}
	}

	if (find(final_state.begin(), final_state.end(), now_ptr->getStart()) != final_state.end() && res.size() > 0) {
		cout << start_ << "->";
		for (int32_t i = 0; i < res.size() - 1; i++) {
			cout << res[i] << "->";
		}
		cout << res[res.size() - 1] << endl << "True";
		return 0;
	}
	cout << "False";

	for (int i = 0; i < res.size(); i++) {
		delete nodes_[i];
	}
	return 0;
}
```
