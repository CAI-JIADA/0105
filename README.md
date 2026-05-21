## 第三份
# Kruskal and Prim 

## 解題說明

本題要求建立一個類別 `Graph` 的物件，並實作兩種最經典的最小生成樹演算法：

1. **Kruskal's Algorithm**
2. **Prim's Algorithm**

### 解題策略
整個程式的解題策略採用「物件導向的繼承機制」將圖片中的抽象基底類別 `Graph` 與自訂的圖形資料結構融合，並透過參數傳遞來執行獨立的 MST 演算法：

1. 先建立`Graph`然後使用`MyGraph`並把原本在`main`函式裡的`edgeList`和`adj`當他的成員變數。
2. 在`main`讓使用者輸入資料，建立`MyGraph`物件，最後分別丟給`Prim`和`Kruskal`跑出答案。
3. 在跑`Prim`和`Kruskal`時要確定不會形成循環。

## 程式實作

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
using namespace std;
class Graph
{
public:
	virtual ~Graph() {}
	bool IsEmpty() const { return n == 0; };
	int NumberOfVertices() const { return n; };//傳回圖中的頂點數
	int NumberOfEdges() const { return e; };//傳回圖中的邊數
	virtual int Degree(int u) const = 0;//傳回與頂點u關聯的邊數量
	virtual bool ExistsEdge(int u, int v) const = 0;//當圖存在邊時傳回t
	virtual void InsertVertex(int v) = 0;//將頂點v插入圖中 v沒有關聯邊
	virtual void InsertEdge(int u, int v) = 0;//將邊(u,v)插入圖
	virtual void DeleteVertex(int v) = 0;//刪除v及其所有關聯邊
	virtual void DeleteEdge(int u, int v) = 0;//從圖中刪除邊(u,v)
private:
	int n;//頂點數量
	int e;//邊數
public:
	Graph(int numV = 0) 
	{
		n = numV;
		e = 0;//一開始的邊一定是0
	}
protected:
	void setN(int numV) { n = numV; }//設定點數量
	void setE(int numE) { e = numE; }//設定邊數量
	void addE() { e++; }//邊的數量+1
	void delE() { e--; }//邊的數量-1
};


struct Edge//定義邊
{
	int u, v, weight;
};
struct PrimEdge
{
	int now, weight;//now=目前頂點編號
	bool operator>(const PrimEdge& other) const {
		return weight > other.weight;//讓每次pop時都能推出權重最小的邊
	}
};
bool Edges(Edge a, Edge b)//邊由小到大
{
	return a.weight < b.weight;
}
class UF//union-find
{
public:
	vector<int> p;//p=根節點
	UF(int n)//初始化
	{
		p.resize(n);
		for (int i = 0; i < n; i++) p[i] = i;
	}
	int find(int i)//找根節點
	{
		if (p[i] == i)return i;
		else return p[i] = find(p[i]);//掛在根節點下
	}
	bool merge(int i, int j)//合併兩個集合
	{
		int I = find(i);
		int J = find(j);
		if (I != J) {
			p[I] = J;//對方為根節點
			return true;//成功合併
		}
		else return false;//本來就同屬一個集合，會造成循環
	}
};


class MyGraph : public Graph//繼承Graph的衍生類別
{
public:
	vector<Edge> edgeList;
	vector<vector<pair<int, int>>> adj;
	// 建構子：初始化頂點數量，並同步呼叫基底類別的建構子
	MyGraph(int numV) : Graph(numV) {
		adj.resize(numV);
	}
	void InsertEdgeWithWeight(int u, int v, int w)//新增一個有權重的插入邊函式
	{
		edgeList.push_back({ u, v, w });
		adj[u].push_back({ v, w });
		adj[v].push_back({ u, w });
		addE();
	}
	//Graph實作
	virtual int Degree(int u) const override 
	{
		return adj[u].size();
	}
	virtual bool ExistsEdge(int u, int v) const override 
	{
		for (auto& edge : adj[u]) 
		{
			if (edge.first == v) return true;
		}
		return false;
	}
	virtual void InsertVertex(int v) override 
	{
		if (v >= NumberOfVertices()) //若插入的點超出範圍則擴展
		{
			setN(v + 1);
			adj.resize(v + 1);
		}
	}
	virtual void InsertEdge(int u, int v) override 
	{
		InsertEdgeWithWeight(u, v, 1); //預設權重=1
	}
	virtual void DeleteVertex(int v) override 
	{
		setE(NumberOfEdges() - adj[v].size());
		adj[v].clear();
	}
	virtual void DeleteEdge(int u, int v) override 
	{
		delE();
	}
};
//Kruskal
void Kruskal(int n, vector<Edge>& edges)
{
	sort(edges.begin(), edges.end(), Edges);//按照邊權重排序
	UF u(n);
	vector<Edge> mst;//儲存最小生成樹
	int mCost = 0;//用來存總權重
	for (Edge& e : edges) {
		if (u.merge(e.u, e.v)) {//確認會不會形成循環 會的話執行 不會的話跳過這條邊
			mst.push_back(e);//將邊加入最小生成樹
			mCost += e.weight;//把這條邊的權重加進總權重裡
		}
	}
	cout << "\nKruskal結果" << endl;
	cout << "總最小權重:" << mCost << endl;
	for (Edge& e : mst)
	{
		cout << e.u << " -- " << e.v << " == " << e.weight << endl;
	}
}
//Prim
void Prim(int n, int sNode, const vector<vector<pair<int, int>>>& adj)//sNode=起始點 第一層vector=圖中的所有頂點 第二層vector=該頂點可以到達的所有鄰居 pair<int, int>=<鄰居編號, 邊的權重>
{
	vector<bool> Lmst(n, false);//記錄哪些頂點已經被納入最小生成樹
	priority_queue<pair<int, pair<int, int>>, vector<pair<int, pair<int, int>>>, greater<pair<int, pair<int, int>>>> pq;//優先佇列：存儲 (權重, 目標節點, 來源節點)
	int mCost = 0;
	vector<Edge> mst;
	pq.push({ 0, {sNode, -1} });//從起點開始
	while (!pq.empty()) {//還有邊就繼續
		int w = pq.top().first;//w=最低的權重
		int u = pq.top().second.first;//u=目標點
		int pre = pq.top().second.second;//pre=來源點
		pq.pop();
		if (Lmst[u]) continue;//如果u已經在樹裡跳過此次 確保不會造成循環
		Lmst[u] = true;//把點加入最小生成樹
		mCost += w;//加上這條邊的權重
		if (pre != -1) mst.push_back({ pre, u, w });//只要不是起點就記錄這條邊
		for (auto& edge : adj[u]) //走訪鄰居
		{
			int v = edge.first;//鄰居
			int weight = edge.second;//到鄰居的距離
			if (!Lmst[v])
			{
				pq.push({ weight, {v, u} });
			}
		}
	}
	cout << "\nPrim結果" << endl;
	cout << "總最小權重:" << mCost << endl;
	for (Edge& e : mst) cout << e.u << " -- " << e.v << " == " << e.weight << endl;
}
int main() {
	int n, e, s;
	cout << "請輸入頂點數量:";
	cin >> n;
	cout << "請輸入邊的數量:";
	cin >> e;
	cout << "請輸Prim的起始點:";
	cin >> s;
	MyGraph g(n);
	cout << "請輸入每條邊的資訊[起點 終點 權重]:" << endl;
	for (int i = 0; i < e; i++) {
		int u, v, w;
		cout << "第" << i + 1 << " 條邊:";
		cin >> u >> v >> w;
		g.InsertEdgeWithWeight(u, v, w);//呼叫內部的插入函式
	}
	Kruskal(n, g.edgeList);
	Prim(n, s, g.adj);
	return 0;
}
```
## 效能分析

1. **Kruskal 演算法**
時間複雜度： $O(e log e)$
空間複雜度： $O(e)$
2. **Prim 演算法**
時間複雜度： $O(elogv)$
空間複雜度： $O(v + e)$

### 測試案例

| 測試案例 | 輸入參數  | 實際輸出Prim |實際輸出Kruskal |
|----------|---------------|----------|----------|
| 測試一   | <img width="359" height="270" alt="image" src="https://github.com/user-attachments/assets/62ec40d3-839a-4170-84d4-5d280ddfcc2f" /> | <img width="134" height="141" alt="image" src="https://github.com/user-attachments/assets/23e2549a-8680-421d-9782-71bace556d08" />| <img width="157" height="132" alt="image" src="https://github.com/user-attachments/assets/d0a187bd-4e3f-4309-8e95-9af34406e13c" />|
| 測試二   | <img width="328" height="220" alt="image" src="https://github.com/user-attachments/assets/ad8d8c8c-312b-4fdc-91dc-373693411dd1" />| <img width="138" height="130" alt="image" src="https://github.com/user-attachments/assets/efc32582-87f9-4511-9db3-924f756be190" />| <img width="139" height="119" alt="image" src="https://github.com/user-attachments/assets/5ef26b40-d22e-4adc-b9ac-86b2429d0d49" />
 |

