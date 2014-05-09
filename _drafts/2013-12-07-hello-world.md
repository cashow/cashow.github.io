---
layout: post
title:  "Hello World"
date:   2013-12-07 23:42:32
categories: jekyll update
---

You'll find this post in your `_posts` directory - edit this post and re-build (or run with the `-w` switch) to see your changes!
To add new posts, simply add a file in the `_posts` directory that follows the convention: YYYY-MM-DD-name-of-post.ext.

Jekyll also offers powerful support for code snippets:

<pre class="prettyprint">#include &lt;vector&gt;
#include &lt;list&gt;
#include &lt;map&gt;
#include &lt;set&gt;
#include &lt;deque&gt;
#include &lt;stack&gt;
#include &lt;algorithm&gt;
#include &lt;iostream&gt;
#include &lt;cstdio&gt;
#include &lt;queue&gt;
#include &lt;string&gt;
#include &lt;cstring&gt;
#include &lt;cmath&gt;
#include &lt;cstdlib&gt;
#include &lt;ctime&gt;

using namespace std;
const double eps=1e-8;

#define MEM(a) memset(a,0,sizeof(a));
#define MEST(a) memset(a,-1,sizeof(a));
#define FOR(i,n) for(int i=0;i&lt;n;i++)
#define rep(i,n) for(int i=1;i&lt;=n;i++)

#define maxn 201

struct edge
{
    int to,from,next,value,num;         //num&#35760;&#24405;&#30340;&#26159;&#36825;&#26465;&#36793;&#30340;&#32534;&#21495;
} e[maxn*maxn*2];

int level[maxn],pre[maxn],next[maxn],minadd[maxn];
int n,m,ans,cnt;
queue&lt;int&gt;q;

void insert(int from,int to,int value)
{
    e[cnt].to=to;
    e[cnt].from=from;
    e[cnt].next=next[from];
    e[cnt].value=value;
    e[cnt].num=cnt;
    next[from]=cnt++;

    swap(from,to);

    e[cnt].to=to;
    e[cnt].from=from;
    e[cnt].next=next[from];
    e[cnt].value=0;
    e[cnt].num=cnt;
    next[from]=cnt++;
}

bool bfs(int start,int end)               //&#26356;&#26032;level&#24182;&#30830;&#23450;&#26159;&#21542;&#33021;&#21040;&#36798;&#27719;&#28857;
{
    int pos,to;
    q.push(start);
    MEST(level);
    level[start]=0;
    while(!q.empty())
    {
        pos=q.front();
        q.pop();
        for(int id=next[pos];id!=-1;id=e[id].next)
        {
            to=e[id].to;
            if(e[id].value&gt;0&amp;&amp;level[to]==-1)
            {
                q.push(to);
                level[to]=level[pos]+1;
            }
        }
    }
    return level[end]!=-1;              //&#33021;&#21040;&#36798;end&#21017;&#36820;&#22238;true&#65292;&#19981;&#33021;&#21017;&#36820;&#22238;false
}

void dfs(int start,int end)
{
    int pos=start,to,change,minn=-1,minpos,id;
    while(1)                            //&#38750;&#36882;&#24402;dfs
    {
        bool flag=false;
        for(id=next[pos];id!=-1;id=e[id].next)      //&#25214;&#19968;&#26465;&#21487;&#20197;&#36208;&#30340;&#36335;
        {
            int i=e[id].to;
            if(level[i]==level[pos]+1&amp;&amp;e[id].value&gt;0)
            {
                flag=true;
                to=i;
                break;
            }
        }

        if(flag)                            //&#25214;&#21040;&#19968;&#20010;&#21487;&#20197;&#36208;&#30340;&#36335;&#24452;
        {
            pre[to]=id;                     //pre[to]&#35760;&#24405;&#30340;&#26159;pos-&gt;to&#36825;&#26465;&#36793;&#30340;id
            if(minadd[pos]==-1)             //minadd[pos]&#21462;&#26368;&#23567;&#12290;&#21040;&#36798;end&#26102;minadd[end]&#20026;&#22686;&#24191;&#36335;&#21487;&#21152;&#30340;&#26368;&#22823;&#27969;&#37327;
                minadd[to]=e[id].value;
            else
                minadd[to]=min(minadd[pos],e[id].value);
            pos=to;			                //&#20174;&#25214;&#21040;&#30340;&#33410;&#28857;&#24320;&#22987;&#22312;&#23618;&#27425;&#22270;&#37324;&#25214;&#36335;
            if(pos==end) 	                //&#25214;&#21040;t&#21518;&#65292;&#22686;&#24191;
            {
                ans+=minadd[end];           //minadd[end]:&#22686;&#24191;&#36335;&#21487;&#21152;&#30340;&#26368;&#22823;&#27969;&#37327;
                change=minadd[end];
                int id=pre[end];
                while(1)              //&#19968;&#30452;&#20174;&#27719;&#28857;&#22238;&#36864;&#21040;&#28304;&#28857;&#65292;&#26356;&#26032;&#27599;&#26465;&#36335;&#30340;&#23481;&#37327;
                {
                    minadd[e[id].to]-=change;
                    e[id].value-=change;
                    e[id^1].value+=change;
                    if(e[id].value==0)
                        pos=e[id].from;           //u&#22238;&#36864;&#21040;&#23481;&#37327;&#36793;&#20026;0&#30340;&#36335;&#30340;&#28857;&#20043;&#21069;
                    if(pre[e[id].from]==-1)
                        break;
                    id=pre[e[id].from];
                }
            }
        }
        else
        {
            if(pos!=start)	            //&#22914;&#26524;pos!=start&#37027;&#20040;&#36825;&#26465;&#36335;&#36208;&#19981;&#36890;&#30340;&#35805;&#65292;&#20174;pos&#30340;&#19978;&#19968;&#20010;&#33410;&#28857;&#32487;&#32493;&#25214;&#12290;
            {
                level[pos]=-2;          //&#30456;&#24403;&#20110;&#20174;&#23618;&#27425;&#22270;&#37324;&#21024;&#38500;&#36825;&#20010;&#33410;&#28857;&#12290;
                pos=e[pre[pos]].from;           //dfs&#22238;&#28335;
            }
            else                        //&#22914;&#26524;&#20174;&#28304;&#28857;&#25214;&#19981;&#21040;&#22686;&#24191;&#36335;&#65292;&#23601;&#32467;&#26463;&#20102;&#12290;
                return ;
        }
    }
}

int dinic(int s,int t)          //&#27714;&#20174;s&#28857;&#21040;t&#28857;&#30340;&#26368;&#22823;&#27969;&#37327;
{
    ans=0;
    while(bfs(s,t))             //&#27599;&#19968;&#27425;&#20808;&#27714;&#20986;&#26368;&#30701;&#36335;&#24452;,&#24314;&#22909;&#23618;&#27425;&#22270;
    {
        MEST(pre);		        //dfs&#20013;&#65292;&#22914;&#26524;&#38656;&#35201;&#22238;&#28335;&#65292;&#23601;&#22238;&#28335;&#21040;pre&#20013;&#30340;&#33410;&#28857;&#12290;
        MEST(minadd);       //minadd&#37324;&#38754;&#23384;&#30340;&#26159;&#21097;&#20313;&#27969;&#37327;
        dfs(s,t);
    }
    return ans;
}

void ini()
{
    MEST(next);
    cnt=0;
}

int main()
{
    int x,y,z;
    while(scanf(&quot;%d%d&quot;,&amp;m,&amp;n)!=EOF)
    {
        ini();
        FOR(i,m)
        {
            scanf(&quot;%d%d%d&quot;,&amp;x,&amp;y,&amp;z);
            insert(x,y,z);
        }
        printf(&quot;%d\n&quot;,dinic(1,n));
    }
    return 0;
}
</pre>