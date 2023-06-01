<!--
 * @Author: yowayimono
 * @Date: 2023-05-20 17:41:46
 * @LastEditors: yowayimono
 * @LastEditTime: 2023-05-20 17:41:59
 * @Description: nothing
-->
```cpp
#include <iostream>
#include <string>
#include <cmath>
#include <algorithm>

std::string to_string(int val){
    std::string result;
    bool flag=false;
    if(!val){
        return "0";
    }
    if(val<0){
        flag=true;
        val=-val;
    }
    
    while (val>0)
    {
        /* code */
        result += '0' + val % 10;
        val/=10;
    }

    if(flag){
        result+="-";
    }

    std::reverse(result.begin(),result.end());
    return result;
    
}

std::string to_string(double val){
    std::string result;
    bool flag;

    if(!val){
        return "0.0";
    }

    if(val<0){
        flag=true;
        val=-val;
    }

    auto intx = static_cast<int>(val);
    double fx= val-intx;

    result=to_string(intx);

    if(fx>0){
        result+=".";
        while(fx>0.000001){
            fx*=10;
            int d=static_cast<int> (fx);
            result+='0'+d;
            fx-=d;
        }
    }
    if(flag){
        result.insert(result.begin(),'-');
    }
    return result;
}
int main(){
    auto i=520;
    auto x=3.1415826;
    std::cout<<to_string(x)<<'\n';
}
```