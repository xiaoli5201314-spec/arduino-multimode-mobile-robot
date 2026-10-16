# 编译与运行

## 工具版本

| 工具 / 依赖 | 检查配置 |
| --- | --- |
| Arduino CLI | 1.2.2 |
| AVR Boards Core | arduino:avr 1.8.6 |
| Servo | 1.2.2 |
| 目标板 | arduino:avr:uno |

## Arduino IDE

1. 安装 Arduino AVR Boards 和 Servo 库。
2. 选择 Arduino Uno。
3. 打开 `firmware` 中某个模式目录里的同名 `.ino` 文件。
4. 执行验证，连接控制板后选择串口并上传。

例如，打开 [Obstacle_Avoiding_Mode.ino](../firmware/Obstacle_Avoiding_Mode/Obstacle_Avoiding_Mode.ino) 即可编译超声避障模式。目录名与 `.ino` 文件名相同，符合 Arduino Sketch 的组织方式。

## Arduino CLI

```sh
arduino-cli core update-index
arduino-cli core install arduino:avr@1.8.6
arduino-cli lib install Servo@1.2.2
arduino-cli compile --fqbn arduino:avr:uno firmware/Line_Follower_Mode
arduino-cli compile --fqbn arduino:avr:uno firmware/Obstacle_Avoiding_Mode
arduino-cli compile --fqbn arduino:avr:uno firmware/Follow_Me_Mode
arduino-cli compile --fqbn arduino:avr:uno firmware/Sumo_Mode
arduino-cli compile --fqbn arduino:avr:uno firmware/Drawing_Mode
```

上传时以实际串口替换下面的 `COM5`：

```sh
arduino-cli board list
arduino-cli upload --fqbn arduino:avr:uno --port COM5 firmware/Obstacle_Avoiding_Mode
```

Linux 上串口通常采用 `/dev/ttyUSB0` 或 `/dev/ttyACM0` 一类名称，以 `board list` 的输出为准。

## 持续集成

[编译工作流](../.github/workflows/compile.yml) 安装指定版本的 AVR Core 与 Servo，分别编译五个 Sketch。编译检查用于确认目标板代码及依赖可以生成固件；电机方向、传感器阈值和舵机笔位通过 [上板检查](HARDWARE.md) 核对。

## 2026-10-16 编译记录

使用上表版本对五个 Sketch 执行 `--fqbn arduino:avr:uno --warnings all`，全部生成目标固件：

| 模式 | Flash | 全局 RAM |
| --- | --- | --- |
| Drawing | 2538 B | 50 B |
| Follow Me | 2094 B | 11 B |
| Line Follower | 2468 B | 188 B |
| Obstacle Avoiding | 2216 B | 9 B |
| Sumo | 2726 B | 188 B |

`--warnings all` 显示 AVR Core `new.cpp` 中 `nothrow_t tag` 的未使用参数告警；五个 Sketch 均完成编译和链接。

## 源文件核验

`source-manifest.json` 记录原始文件到仓库路径的映射及 SHA-256。五个 Sketch 保持原始代码字节，目录调整用于支持独立编译。
