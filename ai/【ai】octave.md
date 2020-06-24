# 【ai】octave

[TOC]

## 简介
octave 是一种与 matlab 语法兼容的开源编程语言，旨在解决线性和非线性的数值计算问题

提供了方便的互动命令列接口来解决线性与非线性的数值运算问题，并可将计算结果可视化

通常用于数值线性代数、统计分析、以及执行其他数值实验，也可以作为面向批处理的语言自动数据处理

[官方网站](https://www.gnu.org/software/octave/)

## 安装
``` bash
# CentOS Linux release 7.3.1611 (Core)
yum install octave 
```

``` bash
# MacOS X 10.14

# 安装 brew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew install octave 

# 卸载 brew
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"
```

## 指令
基本指令
``` matlab
% 退出
quit
exit
% 命令帮助文档
help rand
% 命令执行，但不进行输出
cmd1;
% 命令顺序执行
cmd1, cmd2, cmd3
% 命令顺序执行，但不进行输出
cmd1; cmd2; cmd3;
% 换行
a = 1 + ... 
2
```
<br>
数字运算
``` matlab
% 加
1 + 2
% 减
3 - 2
% 乘
2 * 3
% 除
4 / 2
% 幂
2 ^ 4
```
<br>
逻辑运算，输入以 `非0/0` 表示 `true/false`，输出以 `1/0` 表示 `true/false` 
``` matlab
% 等于
1 == 1
% 不等于
1 ~= 2
% 与
1 && 1
% 或
1 || 1
% 异或
xor(1, 1)
```
<br>
变量
``` matlab
% 变量赋值
a = 1
% 圆周率
pi
% 变量删除，不指定变量则删除所有变量
clear a
% 显示当前内存中的变量
who
% 显示当前内存中的变量及信息
whos
```
<br>
输出
``` matlab
% 输出
disp(a)
% 格式化字符串
sprintf('value: %d', a)
% 输出短的小数点后位数，默认
format short
% 输出长的小数点后位数
format long
```
<br>
矩阵
``` matlab
% 创建矩阵
A = [1 2; 3 4; 5 6]
% 快速创建行矩阵，起始值:间隔:结束值，间隔默认为 1
A = 1:1:16
A = 1:6
% 创建元素都为 1 的矩阵，ones(行数, 列数)
ones(2,3)
% 创建元素都为 0 的矩阵，zeros(行数, 列数)
zeros(2, 3)
% 创建元素都为随机值的矩阵，rand(行数, 列数)
rand(2, 3)
% 创建元素都为高斯随机值的矩阵，randn(行数, 列数)
randn(2, 3)
% 创建单位矩阵，eye(行列数)
eye(3)
% 输出表示矩阵大小的矩阵
size(A)
% 输出矩阵的行数或者是列数
size(A, 1)
size(A, 2)
% 输出矩阵的较大的行数或者是列数
length(A)

% 矩阵的索引，获取指定元素
A(2, 1)
% 获取指定的行或列
A(1, :)
A(:, 1)
% 指定多个行或列
A([1 3], :)
A(1, [1 3])
% 通过赋值可以改变矩阵的元素
A(1, :) = [1 2 3]
% 为矩阵增加一个矩阵到其右边
A = [A [1; 2; 3]]
% 为矩阵增加一个矩阵到其下边
A = [A; [1 2 3]]
% 获取一个向量包含矩阵的所有元素
A(:)
```
<br>
矩阵计算
``` matlab
% 相乘
A * B
% 元素分别相乘
A .* B
% . 表示元素的计算，如元素分别进行幂
A .^ 2
% 元素分别进行四则运算
A + 2
% 元素分别进行逻辑运算
A < 3
% log2 运算
log(A)
% e 的幂次方运算
exp(A)
% 转置
A'
% 获取包含矩阵每列的最大值
max(A)
% 获取矩阵每列的最小值
min(A)
% 同时获取首个元素的索引
[val, idx] = max(A)
% 返回一个矩阵包含两个矩阵中对应位置值较大的元素
max(A, B)
% 获取包含矩阵每行的最大值
max(A, [], 2)
% 获取包含矩阵中的最大值
max(A(:))
% 获取包含符合条件的元素
find(A<3)
% 获取包含符合条件的元素的行列
[r, c] = find(A<3)
% 返回一个幻方矩阵，即横、竖、对角线相加都为相同的值，magic(行列数)
magic(3)
% 求所有元素的和
sum(A)
% 求矩阵每列或每行的和
sum(A, 1)
sum(A, 2)
% 求所有元素的积
prod(A)
% 对所有元素的向下取整
floor(A)
% 对所有元素的向上取整
ceil(A)
% 垂直翻转
flipud(A)
% 逆矩阵
pinv(A)
% 行列式
det(A)
```
<br>
数据的保存和加载
``` matlab
% 操作目录的命令
pwd
cd
ls
% 加载文件中的数据
load filename
load('filename')
% 存储数据到文件
save filename A
```
<br>
数据绘制
``` matlab
% 输出线形图，plot(x轴, y轴)
plot(A, B)
% 传入一个矩阵，每行表示一个图，x轴为列索引，y轴为元素值
plot(A)
% 保留原图像，可以实现两个图像绘制在一个图中
hold on;
% 设置图像颜色
plot(A, B, 'r')
% 设置 x 或 y 轴说明
xlabel('key')
ylabel('value')
% 设置图像说明，可设置同时多个图像
legend('img1', 'img2')
% 关闭图像说明
legend('off')
% 设置标题
title('title')
% 保存图像
print -dpng 'filename.png'
% 关闭图像
close
% 为图像标号
figure(1);plot(A, B)
% 将图像分成 1*2 的网格，进入第1个网格
subplot(1, 2, 1)
% 设置刻度，axis([x轴起始, x轴结束, y轴起始, y轴起始])
axi([1,2,0.5,1])
% 清除图像
clf;
% 显示矩阵的块型图，显示颜色棒和设置为灰色
imagec(magic(3)), colorbar, colormap gray
% 输出条形图，x轴为元素值，y轴为个数
hist(A)
```
<br>
逻辑控制
``` matlab
% 迭代循环，循环 1 到 10 赋值到 i
for i=1:10,
    A(i) = 2 ^ i;
    % break、continue 可正常使用
end;

% 条件循环，
while i <= 5,
    A(i) = 2 ^ i;
    % break、continue 可正常使用
end;

% 条件控制
if i == 6,
    A(i) = 2 ^ i;
elseif i == 5,
    A(i) = 3 ^ i;
else
    A(i) = 4 ^ i;
end;
```
<br>
函数，通过名为 `funcName.m` 的文件来定义，函数文件放在搜索路径中可直接调用
``` matlab
% 函数文件中的语法

% 单个响应的函数
function y = funcName(x)
y = x ^ 2;

% 多个响应的函数
function [y1, y2] = funcName(x)
y1 = x ^ 2;
y2 = x ^ 3;
```
搜索路径默认包括了当前目录，也可以自定义添加目录到搜索路径
``` matlab
addpath('/root/octave')
```
指向函数的指针
``` matlab
@funcName
```

## 算法用例
实现优化梯度下降算法：

参数：$\theta=[\begin{matrix}\theta_1\\\theta_2\end{matrix}]$

代价函数： $J(\theta)=(\theta_1-5)^2+(\theta_2-5)^2$

偏导数：$\begin{array}\ &\frac{\delta}{\delta\theta_1}J(\theta)=2(\theta_1-5)\\
&\frac{\delta}{\delta\theta_2}J(\theta)=2(\theta_2-5)\end{array}$

``` matlab
% costFunc 函数文件
function [jVal, gradient] = costFunc(theta)
jVal= ((theta(1)-5)^2 + (theta(2)-5)^2)
gradient = zeros(2, 1)
gradient(1) = 2*(theta(1)-5)
gradient(2) = 2*(theta(2)-5)
```

``` matlab
% 执行命令
options = optimset('GradObj', 'on', 'MaxIter', '100');
% theta 需要等于或者高于2维
initTheta = zeros(2, 1)
[optTheta, funnctionVal, exitFlag] = fminunc(@costFunc, initTheta, options)
```
<br>
同样地实现正则化的优化梯度下降算法：

参数：$\theta=[\begin{matrix}\theta_1\\\theta_2\end{matrix}]$

代价函数：$J(\theta)=(\theta_1-5)^2+(\theta_2-5)^2+\frac{\lambda}{2m}\sum^2_{j=1}\theta_j$

偏导数：$\begin{array}\ &\frac{\delta}{\delta\theta_1}J(\theta)=2(\theta_1-5)-\frac{\lambda}{2m}\theta_1 \\
&\frac{\delta}{\delta\theta_2}J(\theta)=2(\theta_2-5)-\frac{\lambda}{2m}\theta_1\end{array}$
<br>

实现神经网络中的梯度偏导数和参数的构造:
``` matlab
% 需要转化为长向量
thetaVec = [Theta1(:); Theta2(:); Theta3(:)];
DVec = [D1(:); D2(:); D3(:)];
```
重新取出 `Theta1`，其维度是 `10*11`：
``` matlab
function [jVal, gradientVec] = costFunc(thetaVec)
Theta1 = reshape(thetaVec(1:110), 10, 11)
```
计算出 $J(\theta)$ 和 $D^{(l)}$，放入 `jVal` 和 `gradientVec`
