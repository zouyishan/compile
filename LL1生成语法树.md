生成语法树的过程比较好理解，在考试1做的LL1文法分析句子流程中，对于输入的句子，将其分析过程中推到所有的产生式存储下来，最终生成语法树。

前面已经做过LL1文法分析句子，所以可以直接将用到的产生式都存下来，主要是如何输出是个问题。
```
E T F i
    B @
  A +
    T F i
      B *
        F i
        B @
    A @
```
开始直想用深搜直接输出答案，但是发现深搜的集合还好弄出来，但是每个字符输出的位置信息很难记录，做了很多深搜的trick，最后放弃了。

最后解决是通过将每个字符的状态存起来，这里的状态包括行和列。定义的结构体如下：
```cpp
struct node {
	char ch;
	int row, column;
};
vector<node> a;
```
深搜将搜到的字符丢到容器a中，将所有字符都搜到以后进行排序输出。

具体的深搜代码如下，主要就是在深搜的基础上**想办法记录行和列的信息**，踩的一个坑是不了解定义，每个产生式只能使用一次。也就是代码里面的要用book数组记录下使用情况，已经使用的产生式就不能用了。
```cpp
// 记录文法的使用情况，已经使用的不能重复使用
bool book[100]; 
// 用于记录当前 要递归文法 的位置
int pos = 0;
string find_str(char ch, int k) {
    // 从k开始找，k的含义是调用此函数的产生式的位置
	for (int i = k; i < ans.size(); i++) {
		if (ans[i][0] == ch && book[i] == false) {
            // 找到了就意味着要进入下一层递归了，pos位置要变，而且当前文法也被使用了。
			pos = i;
			book[i] = true;
			return ans[i];
		}
	}
	return "";
}

int r_ = 0;
void dfs(string str, int col) {
	// fl记录当前文法位置，pos会变
	int fl = pos;
	int count = 1;
	for (int i = 3; i < str.size(); i++) {
        // 压入当前字符
		node t;
		t.ch = str[i];
		t.row = r_;
		t.column = col + 1;
		a.push_back(t);
		string tmp = find_str(str[i], fl + 1);
        // 不为空就进入下一个递归。
		if (tmp != "") {
			dfs(tmp, col + 1);
		}
		count++;
        // 这里也好理解，当前层的递归完了，行数就要考。
		r_++;
	}
}
```
把所有要输出的字符，连同状态信息都收集了就好办了，先根据`row`排序，然后再根据`column`排序，然后输出即可。

其实我们这里不用排序，因为由于我们深搜的特性，`row`和`column`都是有序添加进容器的。所以直接输出即可,以下是代码:
```cpp
// 压入第一个字符。
node f;
f.ch = ans[0][0];
f.row = 0;
f.column = 0;
a.push_back(f);
dfs(ans[0], 0);
cout << a[0].ch;
for (int i = 1; i < a.size(); i++) {
    if (a[i].row == a[i - 1].row) {
        int k = 2 * (a[i].column - a[i - 1].column) - 1;
        cout << setw(k) << " ";
        cout << a[i].ch;
    } else {
        cout << endl << setw(2 * a[i].column) << " " << a[i].ch;
    }
}
```

**我的思路是什么**
这题本来要是说数据，其实就是一个深搜就搞定的问题，但是要按要求输出就必须要用额外的变量来记录一些状态信息。开始一直在搞深搜的逻辑想直接输出，但都不太行。

然后就忘往记录每个字符状态信息方面想，主要就是行和列，然后就定义了node这个结构体，只要在深搜中记录好字符的位置信息，然后往结构体里面塞，塞完排序即可。


最后贴下完整代码：
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <map>
#include <iomanip>

using namespace std;
typedef map<char, vector<char> > _map;
int n;
// 左边出现的VN,VT顺序, e表示VN存在空的
vector<char> vn, vt, e;
_map m, follow;
// 一个文法的select集
map<int, vector<char> > ss;
// 做拓展，不止要给first集用
string s[10], target;
// 推出是空的情况
bool flag;

vector<string> ans;

bool isVN(char c) {
	return c >= 'A' && c <= 'Z';
}

struct node {
	char ch;
	int row, column;
};
vector<node> a;

bool isRepeat(vector<char> a, char c) {
	if (a.size() == 0) {
		return false;
	}
	for (int i = 0; i < a.size(); i++) {
		if (a[i] == c) {
			return true;
		}
	}
	return false;
}

vector<char> findVTList(char c) {
	vector<char> tmp;
	for (int i = 0; i < m[c].size(); i++) {
		if (isVN(m[c][i])) {
			vector<char> temp = findVTList(m[c][i]);
			if (tmp.size() > 1 && tmp[tmp.size() - 1] == '@') {
				tmp.pop_back();
			}
			for (int j = 0; j < temp.size(); j++) {
				if (!isRepeat(tmp, temp[j])) {
					tmp.push_back(temp[j]);
				}
			}
			continue;
		}
		if (!isRepeat(tmp, m[c][i]) && m[c][i] != '|') {
			if (tmp.size() > 1 && tmp[tmp.size() - 1] == '@' && m[c][i - 1] != '|') {
				tmp.pop_back();
			}
			tmp.push_back(m[c][i]);
		}
	}
	if (isRepeat(e, c)) {
		tmp.push_back('@');
	}
	return tmp;
}

bool isEnd(int n, int pos) {
	for (int i = pos; i < s[n].size(); i++) {
		if ((!isVN(s[n][i]) && s[n][i] != '@') || !isRepeat(e, s[n][i])) {
			return false;
		}
	}
	return true;
}

bool isLL1() {
	for (int i = 0; i < n; i++) {
		for (int j = i + 1; j < n; j++) {
			if (s[j][0] == s[i][0]) {
				for (int k = 0; k < ss[j].size(); k++) {
					if (isRepeat(ss[i], ss[j][k])) {
						return false;
					}
				}
			}
		}
	}
	return true;
}

string findTable(char nno, char t) {
	for (int i = 0; i < n; i++) {
		if (s[i][0] == nno && isRepeat(ss[i], t)) {
			return s[i];
		}
	}
	return "END";
}

bool book[100]; 
int pos = 0;
string find_str(char ch, int k) {
	for (int i = k; i < ans.size(); i++) {
		if (ans[i][0] == ch && book[i] == false) {
			pos = i;
			book[i] = true;
			return ans[i];
		}
	}
	return "";
}

int r_ = 0;
void dfs(string str, int col) {
	// 记录当前文法位置 
	int fl = pos;
	int count = 1;
	for (int i = 3; i < str.size(); i++) {
		node t;
		t.ch = str[i];
		t.row = r_;
		t.column = col + 1;
		a.push_back(t);
		string tmp = find_str(str[i], fl + 1);
		if (tmp != "") {
			dfs(tmp, col + 1);
		}
		count++;
		r_++;
	}
}

int main(int argc, char *argv[]) {
	cin >> n;
	int t = n;
	while (n--) {
		cin >> s[n];
		// 筛选第一个
		for (int i = 0; i < s[n].length(); i++) {
			if (isVN(s[n][i]) && !isRepeat(vn, s[n][i])) {
				vn.push_back(s[n][i]);
				continue;
			}
			if (!isVN(s[n][i]) && i != 1 && i != 2 && !isRepeat(vt, s[n][i]) && s[n][i] != '@') {
				vt.push_back(s[n][i]);
			}
		}
		if (s[n][3] == '@') {
			if (!isRepeat(e, s[n][0])) {
				e.push_back(s[n][0]);
			}
		}
	}
	cin >> target;
	n = t;
	int z = 3;
	while (z--) {
		while (n--) {
			int k = 3;
			vector<char> tmp = m[s[n][0]];
			while (k < s[n].size() && s[n][k] != '@') {
				if (!isRepeat(tmp, s[n][k])) {
					tmp.push_back(s[n][k]);
				}
				if (isRepeat(e, s[n][k])) {
					k++;
					if (k == s[n].size()) {
						if (!isRepeat(e, s[n][0])) {
							e.push_back(s[n][0]);
							tmp.push_back('|'); // 结束符
						}
					}
				} else {
					m[s[n][0]] = tmp;
					tmp.push_back('|');
					break;
				}
				m[s[n][0]] = tmp;
			}
		}
		n = t;
	}
	n = t;
	
	for (int i = 0; i < vn.size(); i++) {
		vector<char> tmp = findVTList(vn[i]);
		vector<char> res;
		for (int i = 0; i < vt.size(); i++) {
			if (isRepeat(tmp, vt[i])) {
				res.push_back(vt[i]);
			}
		}
		if (tmp[tmp.size() - 1] == '@') {
			res.push_back('@');
		}
		m[vn[i]] = res;
	}
	
	cout << "Vt:";
	for (int i = 0; i < vt.size(); i++) {
		if (i != 0) {
			cout << " ";
		}
		cout << vt[i];
	}
	cout << endl << "Vn:";
	for (int i = 0; i < vn.size(); i++) {
		if (i != 0) {
			cout << " ";
		}
		cout << vn[i];
	}
	
	n = t;
	vector<char> tmp;
	tmp.push_back('#');
	follow[s[n - 1][0]] = tmp;
	z = 3;
	while (z--) {
		while (n--) {
			for (int i = 3; i < s[n].size(); i++) {
				// 找first
				if (isVN(s[n][i]) && i < s[n].size() - 1 ) {
					vector<char> temp = follow[s[n][i]];
					if (isVN(s[n][i + 1])) {
						for (int j = 0; j < m[s[n][i + 1]].size(); j++) {
							if (!isRepeat(temp, m[s[n][i + 1]][j])) {
								temp.push_back(m[s[n][i + 1]][j]);
							}
						}
						if (isEnd(n, i + 1)) { // 弄个函数，检查是否能推出空
							for (int j = 0; j < follow[s[n][0]].size(); j++) {
								if (!isRepeat(temp, follow[s[n][0]][j])) {
									temp.push_back(follow[s[n][0]][j]);
								}
							}
						}
					} else {
						if (!isRepeat(temp, s[n][i + 1])) {
							temp.push_back(s[n][i + 1]);
						}
					}
					follow[s[n][i]] = temp;
				}
				if (isVN(s[n][i]) && i == s[n].size() - 1) {
					vector<char> temp = follow[s[n][s[n].size() - 1]];
					for (int i = 0; i < follow[s[n][0]].size(); i++) {
						if (!isRepeat(temp, follow[s[n][0]][i])) {
							temp.push_back(follow[s[n][0]][i]);
						}
					}
					follow[s[n][s[n].size() - 1]] = temp;
				}
			}
		}
		n = t;
	}
	
	for (int i = 0; i < vn.size(); i++) {
		vector<char> tmp;
		for (int j = 0; j < vt.size(); j++) {
			if (isRepeat(follow[vn[i]], vt[j])) {
				tmp.push_back(vt[j]);
			}
		}
		if (isRepeat(follow[vn[i]], '#')) {
			tmp.push_back('#');
		}
		follow[vn[i]] = tmp;
	}
	
	n = t;
	while (n--) {
		int k = 3;
		char f = s[n][k];
		if (f != '@') {
			vector<char> res;
			while (k < s[n].size() && isVN(f)) {
				for (int i = 0; i < m[f].size(); i++) {
					if (m[f][i] != '@') {
						res.push_back(m[f][i]);
					}
				}
				if (isRepeat(e, f) && k != s[n].size() - 1) {
					f = s[n][++k];
					if (isVN(f)) {
						continue;
					}
				}
				if (!isVN(f)) {
					res.push_back(f);
					f = 'A';
				}
				
				if (isRepeat(e, f)) {
					res.push_back('@');
				}
				
				ss[n] = res;
				break;
			}
			if (!isVN(f)) {
				vector<char> tmp;
				tmp.push_back(f);
				ss[n] = tmp;
			}
		} else {
			vector<char> tmp;
			tmp.push_back('@');
			ss[n] = tmp;
		}
	}
	
	n = t;
	while (n--) {
		vector<char> temp = ss[n];
		if (ss[n][ss[n].size() - 1] == '@') {
			temp.pop_back();
			for (int i = 0; i < follow[s[n][0]].size(); i++) {
				if (!isRepeat(temp, follow[s[n][0]][i])) {
					temp.push_back(follow[s[n][0]][i]);
				}
			}
		}
		ss[n] = temp;
	}

	n = t;
	while (n--) {
		vector<char> temp;
		for (int i = 0; i < vt.size(); i++) {
			if (isRepeat(ss[n], vt[i])) {
				temp.push_back(vt[i]);
			}
		}
		if (isRepeat(ss[n], '#')) {
			temp.push_back('#');
		}
		ss[n] = temp;
	}
	
	n = t;
	vt.push_back('#');
	cout << endl;
	
	vector<char> analyseStack;
	analyseStack.push_back('#');
	analyseStack.push_back(vn[0]);
	target += '#';
	cout << setw(15) << "analyseStack" << setw(15) << "remainStr" << setw(15) << "progress" << endl;
	
	bool flag = false;
	while (true) {
		cout << setw(15 - analyseStack.size()) << " ";
		for (int i = 0; i < analyseStack.size(); i++) {
			cout << analyseStack[i];
		}
		cout << setw(15) << target;
		
		if (isVN(analyseStack.back())) {
			string res = findTable(analyseStack.back(), target[0]);
			if (res == "END") {
				cout << setw(15) << "error" << endl;
				break;
			}
			cout << setw(15) << res << endl;
			ans.push_back(res);
			
			analyseStack.pop_back();
			for (int i = res.size() - 1; i >= 3; i--) {
				if (res[i] != '@') {
					analyseStack.push_back(res[i]);
				}
			}
		} else {
			if (analyseStack.back() == target[0]) {
				if (target[0] == '#') {
					cout << setw(15) << "acc" << endl;
					flag = true;
					break;
				}
				cout << setw(8) << " " << target[0] << " match" << endl;
				analyseStack.pop_back();
				target = target.substr(1);
			} else {
				cout << setw(15) << "error" << endl;
				break;
			}
			
		}
	}
	if (flag) {
		cout << "success!";
	} else {
		cout << "failure!";
	}
	cout << endl;
	node f;
	f.ch = ans[0][0];
	f.row = 0;
	f.column = 0;
	a.push_back(f);
	dfs(ans[0], 0);
	cout << a[0].ch;
	for (int i = 1; i < a.size(); i++) {
		if (a[i].row == a[i - 1].row) {
			int k = 2 * (a[i].column - a[i - 1].column) - 1;
			cout << setw(k) << " ";
			cout << a[i].ch;
		} else {
			cout << endl << setw(2 * a[i].column) << " " << a[i].ch;
		}
	}
	return 0;
}

```