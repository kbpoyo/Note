### 什么是DFA

> DFA: 确定有限状态自动机，由数量有限的状态组成，在其中一个状态下，可通过属于当前状态的通路去往下一个状态。

### 算法4中的dfa

算法4中的dfa为一个二维数组dfa\[\]\[\], 数组纵坐标对应状态机的每一个状态，横坐标对应每一个状态的通路，而数组的值对应着当前状态下通过某一通路能去往的下一个状态。

> 模式串pat：ABABAC

由模式串pat可以构造出状态机dfa如下图：
![](http://imgs.kbpoyo.top/imgs/algorithm_4_202211281335872.png)

其中，dfa\['B'\]\[1] = 2，表示在 *1状态* 下通过通路 *'B'* 可以去往下一个状态 *2* 。

>接下来用*Zn* 表示状态 *n* ，*Cn* 表示状态n对应的通路，而此时通路为ascii字符，*Xn* 表示状态n的重启态(待会说明)，*T<sub>i</sub>* 表示当前与模式串pat匹配的第i个文本字符

### dfa的工作流程

在文本中查找模式串pat，文本字符 *T<sub>i</sub>* 首先与pat\[0\]相比较，若相同，则继续比较 *T*<sub>i+1</sub> 和pat\[1\]，若不同，则 *T*<sub>i+1</sub> 继续与pat\[0\]相比较， 这一个过程用自动机dfa来实现则是，pat的0状态 *Z*<sub>0</sub> 通过通路 *T*<sub>i</sub> 去往下一个状态， 若 *T*<sub>i</sub> = pat\[0\]， 则匹配成功，进入下一个状态dfa\[pat\[0\]\]\[0\] = 1，即 状态*Z*<sub>1</sub> ， 若 *T*<sub>i</sub> != pat\[0\], 则匹配失败，进入下一个状态*Z*<sub>0</sub> ，每一次匹配过后，无论是否匹配成功*T*<sub>i</sub> 都变为 *T*<sub>i + 1</sub> ，dfa进入下一个状态，这样则最多只需要遍历文本字符串一遍则可确定是否查找到pat。

### 什么是重启态

很明显，要想得到线性级别的复杂度，我们不能每次匹配失败后都直接将文本字符回退到一开始与*C*<sub>0</sub> 匹配的字符的下一个字符，因为当*Cn* 匹配失败时，我们其实是知道当前匹配失败的文本字符*T* 的前n个字符为*C*<sub>0</sub> ~ *Cn-1* ，按完全回退的方法来操作，我们需要将pat整体右移一位，继续从*Z*<sub>0</sub>对应的字符*C*<sub>0</sub> 开始比较，但此时我们已知了接下来所要比较的文本字符序列为*C*<sub>1</sub> ~ *C*<sub>n-1</sub>（因为整体右移一位，所以忽略*C*<sub>0</sub>），即*C*<sub>0</sub> 与*C*<sub>1</sub> 比较即 *Z*<sub>0</sub> 通过通路*C*<sub>1</sub> 进入下一个状态继续与*C*<sub>2</sub>进行比较，一直到将*C*<sub>n-1</sub> 比较完毕，便进入了一个状态，将它命名为*X*<sub>n</sub> ，因为此时作为*X*<sub>n</sub>的通路的文本字符为上一次比较的失败字符*T* ，所以我们可以通过*Xn*的通路*T* 进入下一个状态，而有没有一种方法让我们在第一次*Cn* 与 *T*匹配失败时，就直接进入*Xn*，其实将*Zn* 在匹配失败时的通路去向设置为与*Xn*对应通路的去向相同即可，此时*Xn*的效果就好像状态机因匹配失败而重启到了另一个状态一样，所以将*Xn* 称为*Zn*的重启态。

匹配失败：
![](http://imgs.kbpoyo.top/imgs/202211281358193.png)

完全回退继续挨个比较：
![](http://imgs.kbpoyo.top/imgs/202211281359866.png)

利用重启态*X*<sub>4</sub> = *Z*<sub>2</sub> 与失败字符 *'A'* 比较进入状态*Z*<sub>3</sub> 与下一个字符 *'C'* 继续比较：
![](http://imgs.kbpoyo.top/imgs/202211281359183.png)


### 如何找到重启态

**为什么要找到重启态**
为了将每一个状态*Zn*对应的失败字符的通路去向都赋值（成功时进入*Zn+1*即可），我们需要找到重启态*Xn*，将*Zn*对应的失败字符的通路去向设置为与*Xn*对应的相同字符的通路去向相同即可。

**怎样找到重启态**
关于重启态的具体定义我感觉书上说得不太清楚，要想理解如何找到一个状态*Zn* 的重启态*Xn* 我认为需要理解重启态究竟是怎么被定义出来的，以下只是我个人的理解：

各状态*Zn*对应的字符*Cn*: 
![](http://imgs.kbpoyo.top/imgs/202211281710169.png)


首先看*Z*<sub>0</sub>，当pat第一个字符就匹配失败时，我们直接将pat右移一位，并从文本字符的下一个字符与pat第一个字符比较即可，即*Z*<sub>0</sub>的通路去向是确定的，只有当匹配成功时，*Z*<sub>0</sub> 进入 *Z*<sub>1</sub> 否则进入*Z*<sub>0</sub> ，可以理解为*Z*<sub>0</sub> 没有重启态，或者重启态就是自己，因为这就是状态机的初始状态。

*Z*<sub>0</sub> 匹配失败进入*Z*<sub>0</sub> 继续匹配下一位: 
![](http://imgs.kbpoyo.top/imgs/202211281722332.png)


再来看*Z*<sub>1</sub> ，当*C*<sub>1</sub>匹配失败时，我们已知此时失败字符*T*<sub>i</sub> 的前一个字符为*C*<sub>0</sub> ，因为pat整体至少右移一位，所以已知的唯一一个文本字符会被忽略，此时将文本字符完全回退，再前进一位， 其实就相当于没有移动，重新开始匹配的起点就是字符*T*<sub>i</sub> ，所以*Z*<sub>0</sub> 通过通路*T*<sub>i</sub> 进入下一个状态，此时便可以把*Z*<sub>0</sub> 看作*Z*<sub>1</sub>的重启态*X*<sub>1</sub>。

*Z*<sub>1</sub>匹配失败，通过重启态*X*<sub>1</sub> = *Z*<sub>0</sub> 的通路 *'C'* 进入状态*Z*<sub>0</sub>与下一个字符 *'A'* 比较 
![](http://imgs.kbpoyo.top/imgs/202211281756318.png)

*Z*<sub>1</sub>匹配失败，通过重启态*X*<sub>1</sub> = *Z*<sub>0</sub> 的通路 *'A'* 进入状态*Z*<sub>1</sub> 与下一个字符 *'A'* 比较 
![](http://imgs.kbpoyo.top/imgs/202211281752544.png)


最后看任意状态*Zn*(n > 1) ， 当*Cn*与*T*<sub>i</sub>匹配失败时，如前所述，已知的文本序列为*C*<sub>1</sub>~*C*<sub>n-1</sub>
将文本字符完全回退，再前进一位，此时*Z*<sub>0</sub> 通过通路*C*<sub>1</sub> 进入下一个状态再继续通过通路*C*<sub>2</sub>进入下一个状态，再继续通过通路*C*<sub>3</sub>进入再下一个状态，以此往复，直到最后通过通路*C*<sub>n-1</sub>到达某一状态，而这一状态直接通过通路*T*<sub>i</sub> 进入下一状态，完成对文本字符*T*<sub>i</sub>的匹配，继续匹配*T*<sub>i+1</sub>。

我们可以发现，*Z*<sub>0</sub>为*Z*<sub>1</sub>的重启态*X*<sub>1</sub> ，相当于*X*<sub>1</sub> 通过通路*C*<sub>1</sub>进入下一状态，而我们可以就把进入的这一个状态定义为*Z*<sub>2</sub>的重启态*X*<sub>2</sub>，相当于*X*<sub>2</sub>再通过通路*C*<sub>2</sub> 进入*X*<sub>3</sub>，以此往复直到*Z*<sub>n-1</sub> 的重启态*X*<sub>n-1</sub> 通过通路*C*<sub>n-1</sub>进入*Z*<sub>n</sub>的重启态*X*<sub>n</sub>，这样来看重启态的获取其实是一个迭代的过程 ，即当前状态的重启态都可以由以前的状态的重启态得到，而*Z*<sub>0</sub> 的重启态*X*<sub>0</sub> = *Z*<sub>0</sub>，且*Z*<sub>0</sub>状态确定，所以*X*<sub>n</sub> 可求得的。

> **重启态的公式定义**
> *X*<sub>n</sub> = dfa\[*C*<sub>n-1</sub>\]\[*X*<sub>n-1</sub>\]          (n > 1)
> *X*<sub>1</sub> = *Z*<sub>0</sub> ，*X*<sub>0</sub> = *Z*<sub>0</sub>
> *Z*<sub>0</sub> : dfa\[pat\[0\]\]\[0\] = 1，dfa\[pat\[1~length-1\]\]\[0\] = 0

其实重启态就是完全回退后，从*Z*<sub>0</sub> 开始再遍历一遍已知的*C*<sub>1</sub>~*C*<sub>n-1</sub>序列进入的状态，只是这一状态我们可以不用通过回退遍历来获取，而在迭代的过程中一个一个的获取，并将对应状态的失败通路去向设置为重启态相同通路的去向。



### 代码实现


**code**
``` java
public class KMP {  
    private String pat;//模式字符串  
    private int[][] dfa;//自动机 
    
    public KMP(String pat) {  
        this.pat = pat;  
        int M = pat.length();  
        int R = 256;//字符集  
        dfa = new int[R][M];//R为匹配的文本字符，M为当前状态  
        dfa[pat.charAt(0)][0] = 1;//z0 状态匹配到的t为c0时，便可以进入z1状态，匹配到其他任何字符都不进入下一个状态，仍为z0状态  
        int x = 0;//x1为1的重启态， x1 = z0  
        for (int j = 1; j < M; j++) {//完善状态机每一个状态下匹配到字符的去向  
            for (int c = 0; c < R; c++) {  
                dfa[c][j] = dfa[c][x]; //匹配失败时，回到重启态, 相当于直接把重启态匹配相同字符的去向拷贝过来  
            }  
  
            dfa[pat.charAt(j)][j] = j + 1;//匹配成功进入下一个状态, 相当于把去向设置为j + 1(也就是下一个状态)  
  
            x = dfa[pat.charAt(j)][x]; //xn 匹配 cn 进入 xn+1， 获取Zn+1的重启态Xn+1  
        }  
  
    }  
  
    public int search(String txt) {  
        int i, j, N = txt.length(), M = pat.length();  
        for (i = 0, j = 0; i < N && j < M; i++) {  
            j = dfa[txt.charAt(i)][j];//依次匹配文本字符  
        }  
  
        if (j == M) return i - M;//匹配成功算出首位置  
        else return N;//未找到，返回文本字符串的结尾  
  
    }  
  
    public static void main(String[] args) {  
        String pat = "hello";  
        String txt = "halkshdliahjfiaehellapfjalisjdlkajhellojadioljwoijdoiahfilsjdflijaslofjalojf";  
        KMP kmp = new KMP(pat);  
        System.out.println("text:\t\t" + txt);  
        int  offset = kmp.search(txt);  
        System.out.print("pattern:\t");  
        for (int i = 0; i < offset; i++) System.out.print(" ");  
        System.out.println(pat);  
    }  
  
}
```

**运行结果**
![](http://imgs.kbpoyo.top/imgs/202211281917000.png)


