---
title: PID Controller的descrete form
date: 2017-11-10 00:08:18
tags:
- PID
- Control System
- descrete
- robotics
categories:
- Control System
---

PID Controller在各系的自動控制課一定都會講到，我大學時是在大三的時候上到這門課，大致而言學校的自控課程都會先從laplace transform開始教，然後教一些分析系統穩定度的方法，然後會以一些基本的控制模型來講解，PID就是其中一個。

然而不曉得是因為課程安排的關係還是老師太混，我們並沒有教到這套基本控制理論的離散版本，如Discrete PID Controller，z-Transform等等，當時老師又出了程式作業，要求我們要使用樂高機器人來實做倒單擺控制，這對當時的我造成了相當大的疑惑，雖然老師有教程式該怎麼寫，但沒解釋太多，直到最近我才重新理解了一遍PID Controller的discret form，並瞭解到連續系統必須要進行離散化才能在電腦上實做。

PID詳細的理論講解我還在理解整理中，可能之後才會補上，這篇就直接從連續的PID公式講起。

{% raw %}
PID的input為量測值和目標值的誤差$e(t)$，分別對誤差進行比例、積分、微分後以一定的比例($K_p K_i K_d$)組合:

$$
u(t) = K_p e(t) + K_i \int_0^t e(\tau) d\tau + K_d \frac{de(t)}{dt}
$$

其中積分和微分是連續的操作，必須進行離散化，假設系統的取樣時間是$\Delta t$，則積分的離散形式為：

$$
\int_0^{t_n} e(\tau) d\tau = \sum_{i=1}^n e[i] \Delta t = \Delta t \sum_{i=1}^n e[i]
$$

微分（又稱爲有限差分）為：

$$
\frac{de(t_n)}{dt} = \frac{e[n]-e[n-1]}{\Delta t}
$$

帶回原式：

$$
u[n] = K_p e[n] + K_i \Delta t \sum_{i=1}^n e[i] + K_d \frac{e[n]-e[n-1]}{\Delta t}
$$

這時已經可以實做了，需要兩個變數來儲存歷史資料，一個是積分項的$\sum_{i=1}^n e[i]$，一個是微分項的$e[n-1]$，再加上$e[n]$一共三個變數，另外有四個常數，除了本來的三個K值以外還有取樣時間$\Delta t$（有些地方會寫成$T_s$）。
{% endraw %}

總而言之，我對於我們自動控制沒學離散系統感到很困惑，只有學連續系統可能可以分析一些連續的環境，但如果要用電腦來實做，沒有離散的概念是完全不行的，我直到上蘇老師的訊號與系統才開始有這概念。但自控貌似一直都不是本系強項就是了...

以上，如果我有空再補上Arduino的範例或pseudo code，嗯，有空的話（？
