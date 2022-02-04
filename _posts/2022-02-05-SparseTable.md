# emaxx 번역 작업

- sparse table
    
    # 희소배열(Sparse Table)
    
    ### 소개
    
    희소 배열은 구간 쿼리를 효과적으로 처리할 수 있는 자료구조다. 대부분의 구간 쿼리를 $O(\log N)$에 처리할 수 있지만, 구간 최소/최대 쿼리를 처리할 때는 $O(1)$에 가능하다. 하지만 희소 배열은 오직 쿼리의 대상인 **배열의 값이 바뀌지 않는 상황**에서만 적용 가능하다. 즉, 임의의 두 쿼리 사이에 수정 연산이 있어서는 안되며, 수정 된다면 희소 배열 전체가 다시 계산되어야 한다.
    
    ### 직관적으로 이해해보자
    
    모든 음이 아닌 정수는 $2$의 제곱 꼴의 합으로 표현 가능하다. 즉, 이진수로 표현 가능하다. 예를 들어, $13=(1101)_2=8+4+1$ 과 같이 표현할 수 있다. 임의의 양수 $x$에 대하여 최대 $\lceil \log_2{x}\rceil$ 개의 유한합으로 표현할 수 있다.
    
    따라서 임의의 구간은 $2$의 거듭제곱의 길이를 갖는 구간의 합집합으로 표현할 수 있다. 예를 들어서, $[2,14]$를 표현해보자. 구간의 길이가 $13=(1101)_{2}=8+4+1$이므로 $[2,14]=[2,2+8-1]\cup [10,10+4-1]\cup[14,14+1-1]$과 같다. 이때 임의의 길이의 구간은 최대 $\lceil \log_2{(구간의\ 길이)}\rceil$개의 합집합으로 구성된다. 
    
    결국 희소배열의 핵심은 $2^{n}\ (0\leq n\leq \lfloor \log {(length)} \rfloor)$ 을 길이로 가지는 모든 구간에 대한 쿼리를 **미리** 계산하는 것이다. 그 다음 위에서 설명한 규칙으로 임의의 구간을 여러 개의 구간으로 쪼개어 계산한 뒤 합쳐 원하는 구간 쿼리의 답을 얻는다.
    
    ### Precomputation (전처리)
    
    2차원 배열을 이용해 전처리를 해주자. $st[i][j]$는 길이 $2^{j}$의 구간 $[i,i+2^{j}-1]$을 나타낸다고 하자. $MAXN$이 구간 쿼리를 하고자하는 배열의 최대 길이라고 할 때, 2차원 전처리 배열 $st$는 $MAXN\times (\lfloor \log_{2}{MAXN} \rfloor+1)$만큼의 공간을 필요로 한다. $\left( K=\lfloor \log_{2}{MAXN}\rfloor \right)$
    
    ```cpp
    int st[MAXN][K + 1];
    ```
    
    $[i,i+2^{j}-1]$를 주목해보자. 이 구간을 $[i,i+2^{j-1}-1]\cup [i+2^{j-1},i+2^{j}-1]$로 표현하면 길이 $2^{j-1}$의 구간 두 개의 합으로 볼 수 있어, 점화식으로 나타낼 수 있다.
    
    ```cpp
    for (int i = 0; i < N; i++)
        st[i][0] = f(array[i]);
    
    for (int j = 1; j <= K; j++)
        for (int i = 0; i + (1 << j) <= N; i++)
            st[i][j] = f(st[i][j-1], st[i + (1 << (j - 1))][j - 1]);
    ```
    
    함수 $f()$는 구간에 취하고 싶은 연산을 나타낸다. 따라서, 전처리 연산의 시간복잡도는 $O(N\log{N})$이다. 
    
    ### Range Sum Queries (구간 합 쿼리)
    
    구간합을 구해보자. 이 경우, 위 함수 $f$는 $f(x,y)=x+y$일 것이다. 구현은 아래와 같다. 
    
    ```cpp
    long long st[MAXN][K + 1];
    
    for (int i = 0; i < N; i++)
        st[i][0] = array[i];
    
    for (int j = 1; j <= K; j++)
        for (int i = 0; i + (1 << j) <= N; i++)
            st[i][j] = st[i][j-1] + st[i + (1 << (j - 1))][j - 1];
    ```
    
    쿼리 $[L,R]$에 답해보자. 위 전처리 과정과 매우 흡사하게 구할 수 있다. $j=K$부터 시작해 $2^{j}$의 값이 처음으로 구간의 길이$\left(=R-L+1\right)$보다 작거나 같아질 때까지 $j$값을 감소시키면, 구하고자하는 구간 쿼리의 첫 구간인 $[L,L+2^{j}-1]$을 표현할 수 있다. 그리고 남은  $[L+2^{j},R]$의 구간을 동일한 방법으로 구할 수 있다.
    
    ```cpp
    long long sum = 0;
    for (int j = K; j >= 0; j--) {
        if ((1 << j) <= R - L + 1) {
            sum += st[L][j];
            L += 1 << j;
        }
    }
    ```
    
    ### Range Sum Queries (RMQ, 구간 최소 쿼리)
    
    위에서 봤던 합 쿼리와 다른 점을 생각해보면, 이 구간 최소 쿼리의 아름다움을 느낄 수 있다. 구간의 최소값은 같은 구간을 여러번 계산해도 변하지 않는다. 즉 임의의 구간을 길이가 $2$의 거듭제곱인 겹치는 구간으로 표현해도 전혀 상관없다. 예를 들어서, 구간 $[1,6]$의 최소값은 길이가 $4$인 두 개의 구간 $[1,4]$의 최소값과 $[3,6]$의 최소값 중 더 작은 값임이 자명하다. 따라서, $[L,R]$은 다음과 같이 두 개의 구간만으로 구할 수 있다.
    
    $$
    \min(st[L][j], st[R-2^{j}+1][j])\ \ \ \mathrm{where} \ \ \ j=\log_{2}{(R-L+1)}
    $$
    
    $\log_{2}{(R-L+1)}$의 값을 미리 계산해놓자.
    
    ```cpp
    int log[MAXN+1];
    log[1] = 0;
    for (int i = 2; i <= MAXN; i++)
        log[i] = log[i/2] + 1;
    ```
    
    이제 희소배열을 전처리하자. 이때 함수 $f$는 $f(x,y)=\min(x,y)$다.
    
    ```cpp
    int st[MAXN][K + 1];
    
    for (int i = 0; i < N; i++)
        st[i][0] = array[i];
    
    for (int j = 1; j <= K; j++)
        for (int i = 0; i + (1 << j) <= N; i++)
            st[i][j] = min(st[i][j-1], st[i + (1 << (j - 1))][j - 1]);
    ```
    
    이제 구간 쿼리 $[L,R]$을 다음과 같이 $O(1)$에 계산할 수 있다.
    
    ```cpp
    int j = log[R - L + 1];
    int minimum = min(st[L][j], st[R - (1 << j) + 1][j]);
    ```
    
    ### 더 많은 연산을 지원하는 자료구조
    
    $O(1)$에 구간 쿼리를 해결할 수 있다는 사실은 매력적이지만, 위 방법의 치명적인 단점은 쿼리에 필요한 연산이 멱등법칙을 만족해야한다는 것이다. 가령 RMQ는 $O(1)$에 구할 수 있지만, 같은 방법으로 구간 합 쿼리는 처리할 수 없다. 
    
    결합법칙을 만족하는 연산들에 대한 구간쿼리를 $O(1)$에 처리하는 자료구조로는 Disjoint Sparse Table, Sqrt Tree가 있다. (후술 예정)
    
    ### 연습합시다
    
    - [SPOJ - RMQSQ](http://www.spoj.com/problems/RMQSQ/)
    - [SPOJ - THRBL](http://www.spoj.com/problems/THRBL/)
    - [Codechef - MSTICK](https://www.codechef.com/problems/MSTICK)
    - [Codechef - SEAD](https://www.codechef.com/problems/SEAD)
    - [Codeforces - CGCDSSQ](http://codeforces.com/contest/475/problem/D)
    - [Codeforces - R2D2 and Droid Army](http://codeforces.com/problemset/problem/514/D)
    - [Codeforces - Maximum of Maximums of Minimums](http://codeforces.com/problemset/problem/872/B)
    - [SPOJ - Miraculous](http://www.spoj.com/problems/TNVFC1M/)
    - [DevSkills - Multiplication Interval](https://devskill.com/CodingProblems/ViewProblem/19)
    - [Codeforces - Animals and Puzzles](http://codeforces.com/contest/713/problem/D)
    - [Codeforces - Trains and Statistics](http://codeforces.com/contest/675/problem/E)
    - [SPOJ - Postering](http://www.spoj.com/problems/POSTERIN/)
    - [SPOJ - Negative Score](http://www.spoj.com/problems/RPLN/)
    - [SPOJ - A Famous City](http://www.spoj.com/problems/CITY2/)
    - [SPOJ - Diferencija](http://www.spoj.com/problems/DIFERENC/)
    - [Codeforces - Turn off the TV](http://codeforces.com/contest/863/problem/E)
    - [Codeforces - Map](http://codeforces.com/contest/15/problem/D)
    - [Codeforces - Awards for Contestants](http://codeforces.com/contest/873/problem/E)
    - [Codeforces - Longest Regular Bracket Sequence](http://codeforces.com/contest/5/problem/C)
    
    오탈자, 오류 및 기타 첨언은 akim9905@gmail.com 으로 연락주세요.
