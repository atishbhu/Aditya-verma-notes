Here are the **deep-dive, implementation-grade technical notes** for Aditya Verma’s Recursion series. These notes detail exact step-by-step logic, code structures, base conditions, and execution flows for every problem in the playlist.

## **Part 1: High-Level Mental Models & Frameworks**

### **1\. Identify When to Use Recursion vs. DP vs. Backtracking**

                       Is it Recursion?  
                              │  
    ┌─────────────────────────┴─────────────────────────┐  
    ▼                                                   ▼  
Does it have overlapping subproblems?           Are choices being made?  
    │                                                   │  
    ├─► YES: Dynamic Programming (Memoization/Tabulation)├─► Decision Tree (Input/Output Method)  
    │                                                   │  
    └─► NO: Pure Recursion (IBH Method)                 └─► Need to undo choices on return path?  
                                                            ├─► YES: Backtracking  
                                                            └─► NO: Subsets / Permutations

### **2\. Method 1: IBH (Induction – Base Condition – Hypothesis)**

Use IBH when input size decreases linearly ($N \\rightarrow N-1$), such as in stacks, arrays, linked lists, and trees.

> 1. **Hypothesis:** Design a function signature solve(input) and assume it solves the problem for size $N-1$ correctly.  
> 2. **Induction:** Write the single logic step that converts the answer of size $N-1$ into the answer for size $N$.  
> 3. **Base Condition:** Think of the **smallest valid input** or **first invalid input**. Return a concrete answer without making recursive calls.

### **3\. Method 2: Input / Output Method (Decision Trees)**

Use when choices exist at every step (e.g., include/exclude, pick uppercase/lowercase, place space/no space).

                          \[ Input , Output \]  
                                  │  
                  ┌───────────────┴───────────────┐  
                  ▼                               ▼  
       Option A (Include element)     Option B (Exclude element)  
                  │                               │  
            \[ Ip' , Op+X \]                  \[ Ip' , Op \]

* **Recursive Signature:** void solve(string ip, string op)  
* **Base Case:** if (ip.length() \== 0\) { print(op); return; }  
* **Branching Strategy:** Process ip\[0\], then pass ip.substr(1) as the new input for recursive calls.

## **Part 2: Detailed Problem-by-Problem Notes & Implementations**

### **Pattern A: IBH Problems (Arrays, Stacks, Numbers)**

#### **1\. Sort an Array using Recursion**

* **Goal:** Sort an unsorted array recursively without iterative loops.  
* **Hypothesis:** sort(v) sorts vector v. Thus, sort(v.pop\_back()) sorts the remaining $N-1$ elements.  
* **Induction:** Take the popped element val and insert it into its correct position inside the now-sorted vector using a helper function insert(v, val).

C++  
void insert(vector\<int\>& v, int temp) {  
    // Base Case for Insertion: Empty vector OR last element \<= temp  
    if (v.size() \== 0 || v\[v.size() \- 1\] \<= temp) {  
        v.push\_back(temp);  
        return;  
    }  
    int val \= v\[v.size() \- 1\];  
    v.pop\_back();  
    insert(v, temp); // Recursive insertion call  
    v.push\_back(val);  
}

void sortArray(vector\<int\>& v) {  
    if (v.size() \<= 1) return; // Base Condition  
      
    int temp \= v\[v.size() \- 1\];  
    v.pop\_back();  
    sortArray(v);       // Hypothesis (Sort N-1)  
    insert(v, temp);    // Induction (Insert Nth element)  
}

#### **2\. Sort a Stack using Recursion**

* **Logic:** Same as array sorting, but using stack primitives (top(), pop(), push()).

C++  
void insert(stack\<int\>& st, int temp) {  
    if (st.empty() || st.top() \<= temp) {  
        st.push(temp);  
        return;  
    }  
    int val \= st.top();  
    st.pop();  
    insert(st, temp);  
    st.push(val);  
}

void sortStack(stack\<int\>& st) {  
    if (st.size() \<= 1) return;  
      
    int temp \= st.top();  
    st.pop();  
    sortStack(st);  
    insert(st, temp);  
}

#### **3\. Reverse a Stack using $O(1)$ Extra Space**

* **Goal:** Reverse stack in-place using call stack frame memory.  
* **Hypothesis:** reverse(st) reverses stack of size $N-1$.  
* **Induction:** Put top element temp at the **bottom** of the reversed stack using insertAtBottom(st, temp).

C++  
void insertAtBottom(stack\<int\>& st, int ele) {  
    if (st.empty()) {  
        st.push(ele);  
        return;  
    }  
    int temp \= st.top();  
    st.pop();  
    insertAtBottom(st, ele);  
    st.push(temp);  
}

void reverseStack(stack\<int\>& st) {  
    if (st.size() \<= 1) return;  
      
    int temp \= st.top();  
    st.pop();  
    reverseStack(st);  
    insertAtBottom(st, temp);  
}

#### **4\. Delete Middle Element of a Stack**

* **Problem Definition:** $k \= \\lfloor \\frac{\\text{size}}{2} \\rfloor \+ 1$ (1-indexed position from top).  
* **Base Condition:** if (k \== 1\) { st.pop(); return; }

C++  
void deleteMiddle(stack\<int\>& st, int k) {  
    if (st.empty()) return;  
    if (k \== 1) {  
        st.pop(); // Remove middle element  
        return;  
    }  
      
    int temp \= st.top();  
    st.pop();  
    deleteMiddle(st, k \- 1); // Reduce k toward target  
    st.push(temp);           // Re-insert during unwinding  
}

#### **5\. K-th Symbol in Grammar (LeetCode 779\)**

* **Problem:** Row 1 is 0. Subsequent rows are generated by replacing 0 with 01 and 1 with 10.  
* **Observations:**  
  1. Row $N$ has length $2^{N-1}$.  
  2. **First Half** of Row $N$ is identical to Row $N-1$.  
  3. **Second Half** of Row $N$ is the bitwise complement (\!) of Row $N-1$.

  C++

  int kthGrammar(int n, int k) {  
         if (n \== 1 && k \== 1) return 0; // Base condition  
           
         int mid \= pow(2, n \- 2); // Length of previous row / half of current row  
           
         if (k \<= mid) {  
             return kthGrammar(n \- 1, k);  
         } else {  
             return \!kthGrammar(n \- 1, k \- mid);  
         }  
     }

#### **6\. Tower of Hanoi**

* **Goal:** Move $N$ disks from Source S to Destination D using Auxiliary A.  
* **Steps:**  
  1. Move $N-1$ disks from S to A using D.  
  2. Move $N$-th disk directly from S to D.  
  3. Move $N-1$ disks from A to D using S.

C++  
void solve(int n, int s, int d, int a, long long& count) {  
    count++;  
    if (n \== 1) {  
        cout \<\< "move disk " \<\< n \<\< " from rod " \<\< s \<\< " to rod " \<\< d \<\< endl;  
        return;  
    }  
    solve(n \- 1, s, a, d, count);  
    cout \<\< "move disk " \<\< n \<\< " from rod " \<\< s \<\< " to rod " \<\< d \<\< endl;  
    solve(n \- 1, a, d, s, count);  
}

### **Pattern B: Decision Tree Problems (Subsets, Permutations, Constraints)**

#### **7\. Generate All Subsets / Print Power Set**

* **Choices for input character ip\[0\]:**  
  * Option 1: Exclude ip\[0\] from output.  
  * Option 2: Include ip\[0\] in output.

C++  
void printSubsets(string ip, string op) {  
    if (ip.length() \== 0) {  
        cout \<\< op \<\< " ";  
        return;  
    }  
      
    string op1 \= op;          // Choice 1: Exclude  
    string op2 \= op \+ ip\[0\];   // Choice 2: Include  
      
    string next\_ip \= ip.substr(1);  
      
    printSubsets(next\_ip, op1);  
    printSubsets(next\_ip, op2);  
}

#### **8\. Print Unique Subsets**

* **Problem:** Input string may contain duplicate characters (e.g., "aab").  
* **Fix:** Store results in an ordered std::set\<string\> or unordered\_set\<string\> to auto-deduplicate.

C++  
void solve(string ip, string op, set\<string\>& res) {  
    if (ip.length() \== 0) {  
        res.insert(op);  
        return;  
    }  
    string op1 \= op;  
    string op2 \= op \+ ip\[0\];  
    string next\_ip \= ip.substr(1);  
      
    solve(next\_ip, op1, res);  
    solve(next\_ip, op2, res);  
}

#### **9\. Permutation with Spaces**

* **Input:** "ABC" $\\$rightarrow **Output:** "A\_B\_C", "A\_BC", "AB\_C", "ABC".  
* **Key Insight:** First character ip\[0\] is always pushed to op directly. Decision tree starts from index 1\.

C++  
void solve(string ip, string op) {  
    if (ip.length() \== 0) {  
        cout \<\< op \<\< endl;  
        return;  
    }  
      
    string op1 \= op \+ "\_" \+ ip\[0\]; // Include space  
    string op2 \= op \+ ip\[0\];       // Do not include space  
      
    string next\_ip \= ip.substr(1);  
      
    solve(next\_ip, op1);  
    solve(next\_ip, op2);  
}

// Initial Call:  
// string ip \= "ABC";  
// solve(ip.substr(1), string(1, ip\[0\]));

#### **10\. Permutation with Case Change**

* **Input:** "ab" $\\$rightarrow **Output:** "ab", "aB", "Ab", "AB".

C++  
void solve(string ip, string op) {  
    if (ip.length() \== 0) {  
        cout \<\< op \<\< endl;  
        return;  
    }  
      
    string op1 \= op \+ (char)tolower(ip\[0\]);  
    string op2 \= op \+ (char)toupper(ip\[0\]);  
    string next\_ip \= ip.substr(1);  
      
    solve(next\_ip, op1);  
    solve(next\_ip, op2);  
}

#### **11\. Letter Case Permutation (LeetCode 784\)**

* **Input:** "a1b2" $\\$rightarrow Handle both alphabets (2 choices) and digits (1 choice).

C++  
void solve(string ip, string op, vector\<string\>& res) {  
    if (ip.length() \== 0) {  
        res.push\_back(op);  
        return;  
    }  
      
    if (isalpha(ip\[0\])) {  
        string op1 \= op \+ (char)tolower(ip\[0\]);  
        string op2 \= op \+ (char)toupper(ip\[0\]);  
        string next\_ip \= ip.substr(1);  
        solve(next\_ip, op1, res);  
        solve(next\_ip, op2, res);  
    } else {  
        string op1 \= op \+ ip\[0\]; // Digit: single branch  
        string next\_ip \= ip.substr(1);  
        solve(next\_ip, op1, res);  
    }  
}

#### **12\. Generate Balanced Parentheses (LeetCode 22\)**

* **Given $N$ pairs of parentheses**, generate all valid combinations.  
* **Constraints for Decision Tree:**  
  * open \> 0: We can always add '('.  
  * close \> open: We can add ')' only if remaining close count is strictly greater than remaining open count.

C++  
void solve(int open, int close, string op, vector\<string\>& res) {  
    if (open \== 0 && close \== 0) {  
        res.push\_back(op);  
        return;  
    }  
      
    // Choice 1: Add Open Bracket  
    if (open \!= 0) {  
        string op1 \= op \+ "(";  
        solve(open \- 1, close, op1, res);  
    }  
      
    // Choice 2: Add Close Bracket  
    if (close \> open) {  
        string op2 \= op \+ ")";  
        solve(open, close \- 1, op2, res);  
    }  
}

#### **13\. N-bit Binary Numbers Having More 1s than 0s in All Prefixes**

* **Constraints for Decision Tree:**  
  * Can always add '1'.  
  * Can add '0' **only if** ones \> zeros.

C++  
void solve(int ones, int zeros, int n, string op) {  
    if (n \== 0) {  
        cout \<\< op \<\< " ";  
        return;  
    }  
      
    // Choice 1: Always valid to append '1'  
    string op1 \= op \+ "1";  
    solve(ones \+ 1, zeros, n \- 1, op1);  
      
    // Choice 2: Append '0' only if ones \> zeros condition holds  
    if (ones \> zeros) {  
        string op2 \= op \+ "0";  
        solve(ones, zeros \+ 1, n \- 1, op2);  
    }  
}

#### **14\. Josephus Problem (Game of Death in a Circle)**

* **Problem:** $N$ people standing in a circle ($0$ to $N-1$). Kill every $K$-th person until 1 survivor remains.  
* **Observation:** After killing the $K$-th person, the array indices shift.

C++  
void solve(vector\<int\>& person, int k, int index, int& ans) {  
    if (person.size() \== 1) {  
        ans \= person\[0\]; // Survivor  
        return;  
    }  
      
    // Calculate index of person to be killed  
    index \= (index \+ k) % person.size();  
    person.erase(person.begin() \+ index);  
      
    solve(person, k, index, ans);  
}

int findTheWinner(int n, int k) {  
    vector\<int\> person;  
    for (int i \= 0; i \< n; i++) person.push\_back(i \+ 1); // 1-indexed  
    int ans \= \-1;  
    solve(person, k \- 1, 0, ans);  
    return ans;  
}

## **Part 3: Cheat Sheet & Time Complexities**

| Category | Problem | Time Complexity | Space Complexity (Call Stack) |
| :---- | :---- | :---- | :---- |
| **IBH** | Sort Array / Stack | $\\mathcal{O}(N^2)$ | $\\mathcal{O}(N)$ |
| **IBH** | Reverse Stack | $\\mathcal{O}(N^2)$ | $\\mathcal{O}(N)$ |
| **IBH** | Delete Middle Stack | $\\mathcal{O}(N)$ | $\\mathcal{O}(N)$ |
| **IBH** | K-th Symbol Grammar | $\\mathcal{O}(N)$ | $\\mathcal{O}(N)$ |
| **IBH** | Tower of Hanoi | $\\mathcal{O}(2^N)$ | $\\mathcal{O}(N)$ |
| **Input/Output** | Subsets / Subsequences | $\\mathcal{O}(2^N)$ | $\\mathcal{O}(N)$ |
| **Input/Output** | Permutations (Spaces/Case) | $\\mathcal{O}(2^N)$ | $\\mathcal{O}(N)$ |
| **Input/Output** | Balanced Parentheses | $\\mathcal{O}(C\_N) \\approx \\frac{4^N}{N^{3/2}}$ | $\\mathcal{O}(N)$ |
| **Input/Output** | Josephus Problem | $\\mathcal{O}(N^2)$ (vector erase) | $\\mathcal{O}(N)$ |

[Aditya Verma Recursion Playlist](https://www.google.com/search?q=https://www.youtube.com/watch?kHi1DUhp9kM)  
This video introduces the foundational concepts and identification framework for solving recursive problems as covered across the playlist.
