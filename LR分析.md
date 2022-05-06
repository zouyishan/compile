> tips: 题目描述在后面
嗯，这个题比较复杂，写出来要结合前面的follow集代码，但是做到这的时候follow集忘的差不多了，以前写的代码还有一些小bug，而且代码还很长。而且follow集代码和构建SLR(1)分析表变量还有很多重合的地方。

这个算法的流程大致是这样：

1. 求出SLR(1)项目集规范族：首先原产生式集合拓广文法
2. 求项目集规范族：描述简单点就是 求集合闭包(closure) -> 移进(go) -> 对移进的字符不同进行分类，分类的集合再次求闭包。直到所有集合都不能移进为止。
3. 判断冲突，冲突分为 “移进-归约” “归约-归约”。如果有冲突就找产生式最前面的非终结符的follow集，如果follow集不冲突那就可以解决。
4. 构建SLR(1)分析表：这个比较复杂，直接看图吧：
//TODO 图片
5. 通过SLR(1)分析表，移进规约输入的字符串
//TODO 图片

**我是如何做/思考这道题的**

首先，我还是将这个题归结为模拟题，没有用到什么特别的算法，但是这题和前面的题不同的是，**这个模拟的思路足够复杂(对课下没复习的人来说是这样的)**。

几个重要的数据结构：
* `vector<vector<node> > state`: **顺序**保存项目集的状态。
* `queue<vector<node> > q`: 为了保证项目集的顺序，移进项按照先进先出的规则存放。
* `exam5_res1[100][200]`: 保存Sx的数组
* `exam5_res2[100][200]`: 保存Rx的数组

贴下核心的规约过程的代码：
```cpp
while (true) {
    cout << setw(4) << step++;
    // 记录栈顶
    int p = statusStack[statusStack.size() - 1];
    char oi = remainStr[remainStr.size() - 1];
    // 要输出的字符串。
    string statusStackStr, symbolStackStr, remainStr_, analyseActStr;
    for (int i = 0; i < statusStack.size(); i++) {
        if (i == 0) {
            statusStackStr += to_string(statusStack[i]);
            continue;
        }
        statusStackStr += " ";
        statusStackStr += to_string(statusStack[i]);
    }
    cout << setw(20) << statusStackStr;
    for (int i = 0; i < symbolStack.size(); i++) {
        symbolStackStr += symbolStack[i];
    }
    for (int i = remainStr.size() - 1; i >= 0; i--) {
        remainStr_ += remainStr[i];
    }
    cout << setw(15) << symbolStackStr << setw(10) << remainStr_;
    
    // 结束
    if (exam5_res1[p][oi] == -1 && exam5_res2[p][oi] == -1) {
        cout << setw(20) << "error";
        break;
    }
    // 移进
    if (exam5_res1[p][oi] != -1) {
        statusStack.push_back(exam5_res1[p][oi]);
        symbolStack.push_back(oi);
        remainStr.pop_back();
        analyseActStr = "shift ";
        analyseActStr += to_string(exam5_res1[p][oi]);
        cout << setw(20) << analyseActStr << endl;
        continue;
    } else if (exam5_res2[p][oi] != -1) {
        // 成功了
        if (exam5_res2[p][oi] == 0) {
            cout << setw(20) << "acc";
            break;
        } else {
            // 这里要归约
            analyseActStr = "reduce by ";
            analyseActStr += ss[exam5_res2[p][oi] - 1];
            cout << setw(20) << analyseActStr << endl;
            int countt = ss[exam5_res2[p][oi] - 1].size() - 3;
            for (int i = 0; i < countt; i++) {
                statusStack.pop_back();
                symbolStack.pop_back();
            }
            symbolStack.push_back(ss[exam5_res2[p][oi] - 1][0]);
            statusStack.push_back(exam5_res1[statusStack[statusStack.size() - 1]][ss[exam5_res2[p][oi] - 1][0]]);
        }
    }
}
```

```cpp
#include <iostream>
#include <vector>
#include <iomanip>
#include <queue>
#include <map>
#include <set>
 
using namespace std;
typedef map<char, vector<char> > _map;
int n;
// 左边出现的VN,VT顺序, e表示VN存在空的

vector<char> vn, vt, e;
vector<char> vvn, vvt;
int exam5_res1[100][200], exam5_res2[100][200];

set<int> rres, kres;
_map m, follow;
struct ans {
    string from;
    char st;
    string to;
}aa[100];
string s[10], ss[100];
//推出是空的情况
bool flag;
map<char, vector<string> > mm;
struct node {
    string str;
    int pos;
}a[100];
vector<vector<node> > state;
queue<vector<node> > q;
string qwer;

bool isVN(char c) {
    return c >= 'A' && c <= 'Z';
}

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
        if (!isRepeat(tmp, m[c][i])) {
            if (tmp.size() > 1 && tmp[tmp.size() - 1] == '@') {
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
 

int isExist(vector<node> c) {
    bool flag = false;
    for (int i = 0; i < state.size(); i++) {
        flag = false;
        if (state[i].size() != c.size()) {
            continue;
        }
        for (int j = 0; j < state[i].size(); j++) {
            if (c[j].str != state[i][j].str || c[j].pos != state[i][j].pos) {
                flag = true;
                break;
            }
        }
        if (!flag) {
            return i;
        }
    }
    return -1;
}

vector<node> getClosure(char c) {
    vector<node> res;
    if (mm.find(c) == mm.end()) {
        return res;
    }
    map<char, vector<string> >::iterator it;
    for (int i = 0; i < mm[c].size(); i++) {
        node tmp;
        tmp.str = mm[c][i];
        tmp.pos = 3;
        res.push_back(tmp);
    }
    
    int m = res.size();
    vector<char> zzz;
    for (int i = 0; i < m; i++) {
        if (isVN(res[i].str[3]) && res[i].str[3] != res[i].str[0]) {
            if (!isRepeat(zzz, res[i].str[3])) {
                zzz.push_back(res[i].str[3]);
                vector<node> k = getClosure(res[i].str[3]);
                res.insert(res.end(), k.begin(), k.end());
            }
        }
    }
    return res;
}

void printPos(node s) {
    bool flag = false;
    for (int i = 0; i < s.str.size(); i++) {
        if (i == s.pos) {
            flag = true;
            cout << "·";
        }
        cout << s.str[i];
    }
    if (!flag) {
        cout << "·";
    }
    cout << endl;
}

void init() {
    for (int i = 0; i < 100; i++) {
        for (int j = 0; j < 200; j++) {
            exam5_res1[i][j] = -1;
            exam5_res2[i][j] = -1;
        }
    }
}

int main(int argc, char *argv[]) {
    init();
    cin >> n;
    int t = n;
    bool flag = false;
    vn.push_back('G');
    int poi = 0;
    while (n--) {
        cin >> s[n];
        ss[poi++] = s[n];
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
    cin >> qwer;
    s[t] += "G->";
    s[t] += vn[1];
    t += 1;
    n = t;
    while (n--) {
        int k = 3;
        vector<char> tmp = m[s[n][0]];
        while (k < s[n].size() && !isRepeat(tmp, s[n][k]) && s[n][k] != '@') {
            if (s[n][k] == s[n][0] && !isRepeat(e, s[n][k])) {
                break;
            }
            tmp.push_back(s[n][k]);
            if (isRepeat(e, s[n][k])) {
                k++;
            } else {
                m[s[n][0]] = tmp;
                break;
            }
            m[s[n][0]] = tmp;
        }
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

    vector<char> tmp;
    tmp.push_back('#');
    follow[s[n - 1][0]] = tmp;
    int z = 3;
    while (z--) {
        while (n--) {
            for (int i = 3; i < s[n].size(); i++) {
                
                if (isVN(s[n][i]) && i < s[n].size() - 1 ) {
                    vector<char> temp = follow[s[n][i]];
                    if (isVN(s[n][i + 1])) {
                        for (int j = 0; j < m[s[n][i + 1]].size(); j++) {
                            if (!isRepeat(temp, m[s[n][i + 1]][j])) {
                                temp.push_back(m[s[n][i + 1]][j]);
                            }
                        }
                        for (int j = 0; j < follow[s[n][i + 1]].size(); j++) {
                            if (!isRepeat(temp, follow[s[n][i + 1]][j])) {
                                temp.push_back(follow[s[n][i + 1]][j]);
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

    n = t - 1;
    for (int i = 0; i < n; i++) {
        if (!isRepeat(vvn, ss[i][0])) {
            vvn.push_back(ss[i][0]);
        }
        for (int j = 3; j < ss[i].size(); j++) {
            if (isVN(ss[i][j]) && !isRepeat(vvn, ss[i][j])) {
                vvn.push_back(ss[i][j]);
            }
            if (!isVN(ss[i][j]) && !isRepeat(vvt, ss[i][j]) && ss[i][j] != '@') {
                vvt.push_back(ss[i][j]);
            }
        }
    }
    
    node x;
    x.str = "G->";
    x.str += vvn[0];
    x.pos = 3;
//  cout << "0." << x.str << endl;
    int step = 1;
    for (int i = 0; i < n; i++) {
//      cout << step++ << "." << ss[i] << endl;
        if (ss[i][ss[i].size() - 1] == '@') {
            mm[ss[i][0]].push_back(ss[i].substr(0, ss[i].size() - 1));
            continue;
        }
        mm[ss[i][0]].push_back(ss[i]);
    }
    cout << "analyseProgress:" << endl;
    vector<node> ttmp;
    vector<node> tt = getClosure(vvn[0]);
    ttmp.push_back(x);
    ttmp.insert(ttmp.end(), tt.begin(), tt.end());
    
    for (int i = 0; i < ttmp.size(); i++) {
        if (ttmp[i].str.size() == ttmp[i].pos) {
            for (int hh = 0; hh < follow[ttmp[i].str[0]].size(); hh++) {
                int nnuumm = -1;
                for (int ggg = 0; ggg < n; ggg++) {
                    if (ss[ggg] == ttmp[i].str || (ss[ggg][3] == '@' && ttmp[i].str.size() == 3)) {
                        nnuumm = ggg;
//                      cout << ggg + 1 << "====" << ttmp[i].str << endl;
                        break;
                    }
                }
                exam5_res2[0][follow[ttmp[i].str[0]][hh]] = nnuumm + 1;
            }
        }
    }
    
    q.push(ttmp);
    int ti = 1;
    int ttt = 0;
    int ppre = -1;
    state.push_back(ttmp);
    // 移进 规约
    while (!q.empty()) {
        vector<node> tmp = q.front();
        ppre++;
        q.pop();
        vector<char> t;
        // 只移进
        for (int i = 0; i < tmp.size(); i++) {
            if (tmp[i].str.size() > tmp[i].pos && !isRepeat(t, tmp[i].str[tmp[i].pos])) {
                t.push_back(tmp[i].str[tmp[i].pos]);
            }
        }
        int ppp = -1;
        // 归约归约冲突
        for (int i = 0; i < t.size(); i++) {
            vector<node> tmp_res, tt;
            vector<char> iii;
            ppp++;
            for (int j = ppp; j < tmp.size(); j++) {
                if (tmp[j].str[tmp[j].pos] == t[i]) {
                    ppp = j;
                    tmp[j].pos++;
                    tmp_res.push_back(tmp[j]);
                    // 两个都有的话，有移进归约冲突
                    if (tmp[j].pos < tmp[j].str.size()) {
                        vector<node> ttttt;
                        if (!isRepeat(iii, tmp[j].str[tmp[j].pos])) {
                            iii.push_back(tmp[j].str[tmp[j].pos]);
                            ttttt = getClosure(tmp[j].str[tmp[j].pos]);
                        }
                        if (ttttt.size() > 0) {
                            tt.insert(tt.end(), ttttt.begin(), ttttt.end());
                        }
                    } else { // 成功 结束
                        
                    }
                }
            }
            
            if (tt.size() > 0) {
                tmp_res.insert(tmp_res.end(), tt.begin(), tt.end());
            }
            
            if (tmp_res.size() > 0 && isExist(tmp_res) == -1) {
                q.push(tmp_res);
                state.push_back(tmp_res);
                exam5_res1[ppre][t[i]] = ti;
                aa[ttt].from = "I" + to_string(ppre);
                aa[ttt].st = t[i];
                aa[ttt].to = "I" + to_string(ti);
                ttt++;
                ti++;
                int tag1 = 0, tag2 = 0;
                for (int kk = 0; kk < tmp_res.size(); kk++) {
                    // 规约
                    if (tmp_res[kk].pos == tmp_res[kk].str.size()) {
                        for (int hh = 0; hh < follow[tmp_res[kk].str[0]].size(); hh++) {
                            int nnuumm = -1;
                            for (int ggg = 0; ggg < n; ggg++) {
                                if (ss[ggg] == tmp_res[kk].str || (ss[ggg][3] == '@' && tmp_res[kk].str.size() == 3)) {
                                    nnuumm = ggg;
                                    break;
                                }
                            }
                            exam5_res2[ti - 1][follow[tmp_res[kk].str[0]][hh]] = nnuumm + 1;
                        }
                        tag1++;
                    } else {
                        tag2++;
                    }
                }
                // 移进规约冲突
                if (tag2 > 0 && tag1 > 0) {
                    rres.insert(ti - 1);
                }
                // 规约规约冲突
                if (tag1 > 1 && tag2 == 0) {
                    kres.insert(ti - 1);
                }
            } else if (tmp_res.size() > 0 && isExist(tmp_res) != -1) {
                // 状态转换
                exam5_res1[ppre][t[i]] = isExist(tmp_res);
                aa[ttt].from = "I" + to_string(ppre);
                aa[ttt].st = t[i];
                aa[ttt].to = "I" + to_string(isExist(tmp_res));
                ttt++;
            }
        }
    }
    
    step = 1;
    vector<int> statusStack;
    vector<char> symbolStack, remainStr;
    statusStack.push_back(0);
    symbolStack.push_back('#');
    for (int i = qwer.size() - 1; i >= 0; i--) {
        remainStr.push_back(qwer[i]);
    }
    
    cout << "step" << setw(20) << "statusStack" << setw(15) << "symbolStack" << setw(10) << "remainStr" << setw(20) << "analyseAct" << endl;
    while (true) {
        cout << setw(4) << step++;
        int p = statusStack[statusStack.size() - 1];
        char oi = remainStr[remainStr.size() - 1];
        string statusStackStr, symbolStackStr, remainStr_, analyseActStr;
        for (int i = 0; i < statusStack.size(); i++) {
            if (i == 0) {
                statusStackStr += to_string(statusStack[i]);
                continue;
            }
            statusStackStr += " ";
            statusStackStr += to_string(statusStack[i]);
        }
        cout << setw(20) << statusStackStr;
        for (int i = 0; i < symbolStack.size(); i++) {
            symbolStackStr += symbolStack[i];
        }
        for (int i = remainStr.size() - 1; i >= 0; i--) {
            remainStr_ += remainStr[i];
        }
        cout << setw(15) << symbolStackStr << setw(10) << remainStr_;
        
        // 结束
        if (exam5_res1[p][oi] == -1 && exam5_res2[p][oi] == -1) {
            cout << setw(20) << "error";
            break;
        }
        // 移进
        if (exam5_res1[p][oi] != -1) {
            statusStack.push_back(exam5_res1[p][oi]);
            symbolStack.push_back(oi);
            remainStr.pop_back();
            analyseActStr = "shift ";
            analyseActStr += to_string(exam5_res1[p][oi]);
            cout << setw(20) << analyseActStr << endl;
            continue;
        } else if (exam5_res2[p][oi] != -1) {
            if (exam5_res2[p][oi] == 0) {
                cout << setw(20) << "acc";
                break;
            } else {
                analyseActStr = "reduce by ";
                analyseActStr += ss[exam5_res2[p][oi] - 1];
                cout << setw(20) << analyseActStr << endl;
                int countt = ss[exam5_res2[p][oi] - 1].size() - 3;
                for (int i = 0; i < countt; i++) {
                    statusStack.pop_back();
                    symbolStack.pop_back();
                }
                symbolStack.push_back(ss[exam5_res2[p][oi] - 1][0]);
                statusStack.push_back(exam5_res1[statusStack[statusStack.size() - 1]][ss[exam5_res2[p][oi] - 1][0]]);
            }
        }
    }
    return 0;
}
```