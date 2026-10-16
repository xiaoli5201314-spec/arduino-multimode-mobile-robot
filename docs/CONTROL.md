# 模式与控制逻辑

各 Sketch 有独立的 `setup()` 与 `loop()`。这里按源码实际分支列出行为，便于结合电机接线阅读。

## 双电机接口

A 路使用 D2 / D4 控制方向、D3 控制占空比；B 路使用 D10 / D11 控制方向、D9 控制占空比。方向 GPIO 的 HIGH / LOW 组合通过 L293D 改变电流方向；PWM 参数为零时停止对应通道。

底盘运动方向同时取决于电机安装位置和接线。调试时先核对单轮方向，再把两轮输出组合映射为前进、后退或转向。

## 红外循迹

入口：[Line_Follower_Mode.ino](../firmware/Line_Follower_Mode/Line_Follower_Mode.ino)。

读取 D12。HIGH 时 A 输出 HIGH / LOW、B 输出 LOW / HIGH；LOW 时 A、B 均输出 HIGH / LOW。两路 PWM 分别为 90 和 110。串口以 9600 波特率输出当前红外读数。

## 超声避障

入口：[Obstacle_Avoiding_Mode.ino](../firmware/Obstacle_Avoiding_Mode/Obstacle_Avoiding_Mode.ino)。

循环中将 Trig 拉高 1000 微秒后拉低，通过 `pulseIn(echoPin, HIGH)` 读取回波，使用 `(duration / 2) / 28.5` 换算距离并存入整数变量。

- 小于 10 cm：两路 PWM 置零，等待 100 ms，然后启动 B 路并等待 350 ms。
- 其余情况：设置两路方向，PWM 分别为 80 / 90。

## 距离跟随

入口：[Follow_Me_Mode.ino](../firmware/Follow_Me_Mode/Follow_Me_Mode.ino)。

测距方法与避障模式一致，PWM 参数为 90 / 100。源码使用多个独立 `if` 分支：

| 距离条件 | 本轮写入动作 |
| --- | --- |
| `< 5` | 两路电机按后退方向输出 |
| `> 5 && < 10` | 两路 PWM 置零 |
| `> 10 && < 20` | 两路电机按前进方向输出 |
| `> 20` | 写入 B 路方向与 PWM，A 路沿用之前的输出 |
| `== 5 / 10 / 20` | 沿用之前的电机输出 |

`pulseIn()` 返回 0 时计算距离为 0，会进入 `< 5` 分支。阅读和上板联调时，把这一路径与真实近距离回波区分开。

## 边界响应 / Sumo

入口：[Sumo_Mode.ino](../firmware/Sumo_Mode/Sumo_Mode.ino)。

红外输入为 HIGH 时，执行停止 200 ms、后退 1000 ms，再切换 B 路方向并保持 400 ms 的序列；LOW 时输出前进组合。串口输出红外电平，PWM 为 90 / 100。

## 舵机绘图

入口：[Drawing_Mode.ino](../firmware/Drawing_Mode/Drawing_Mode.ino)。

初始化舵机到 70°，等待 5 秒后写入 120°。循环中依次调整舵机位置、直行、停车、直行与转向。电机 PWM 为 100 / 110，舵机使用 `Servo.attach(5)`。

该模式把舵机位置和底盘定时动作组合起来。角度与笔位的对应关系应根据实际连杆安装确定。
