# 硬件与接口

## 控制板

ATmega328P 负责输入采样和电机输出，L293D 连接两路直流电机。开发时按 Arduino Uno 选择板型，通过 USB 转串口连接电脑。接线图保存在 [Connections.png](../third_party/RobotControlBoard/Connections.png)。

| 信号 | Arduino 引脚 | 作用 |
| --- | --- | --- |
| Motor A IN1 | D2 | A 路电机方向 |
| Motor A IN2 | D4 | A 路电机方向 |
| Motor A EN | D3 | A 路 PWM |
| Motor B IN1 | D10 | B 路电机方向 |
| Motor B IN2 | D11 | B 路电机方向 |
| Motor B EN | D9 | B 路 PWM |
| 超声 Trig | D6 | 触发测距 |
| 超声 Echo | D5 | 回波脉宽输入 |
| 红外数字输出 | D12 | 循迹 / 边界检测 |
| 绘图舵机信号 | D5 | Servo 库位置输出 |

## 按模式连接

| 模式 | 使用的传感器 / 执行器 |
| --- | --- |
| Line Follower | 两路电机 + D12 红外输入 |
| Obstacle Avoiding | 两路电机 + D6 / D5 超声模块 |
| Follow Me | 两路电机 + D6 / D5 超声模块 |
| Sumo | 两路电机 + D12 红外输入 |
| Drawing | 两路电机 + D5 舵机 |

每次烧录一个模式。D5 在超声模式中接 Echo，在绘图模式中接舵机信号，按所选模式调整接线。电机与传感器的逻辑地需要共地。

## PWM 与 Servo

电机占空比参数传给 `analogWrite()`，取值范围为 0 到 255。源码中 `motorArpm` / `motorBrpm` 是 PWM 参数名称，数值表示占空比等级，不能直接解释成实测转速。

Arduino Uno 的 Servo 库使用 Timer1。绘图示例在 D9 上调用 `analogWrite(..., 110)`，因此应结合 AVR Core 与 Servo 的定时器使用方式核对 D9 输出。该模式上板时先架空底盘，分别检查舵机动作和两路电机输出，再进行轨迹实验。

## 上板检查

1. 架空底盘核对每个电机的正反转方向。
2. 确认 PWM 置零时对应电机停止。
3. 对照串口输出观察红外模块在不同表面上的实际电平。
4. 使用已知距离的静态目标检查超声测距分支。
5. 绘图模式先确定 70° / 120° 对应的实际笔位，再进行落笔实验。
