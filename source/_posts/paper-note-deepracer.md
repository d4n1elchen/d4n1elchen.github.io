---
title: '論文筆記 DeepRacer: Educational Autonomous Racing Platform for Experimentation with Sim2Real Reinforcement Learning'
date: 2020-06-01 00:22:03
tags: [robotics, autonomus car, reinforcement learning, artifical intelligence, neural network]
categories: robotics
---

[PDF](https://arxiv.org/pdf/1911.01562v1.pdf)

## Abstract
這篇主要是講 DeepRacer 的動機、分散式計算架構、模擬環境架構、robust evaluation 的方法、還有他們在上面訓練一個搭載單鏡頭，大小 1/18 的小型自動駕駛車成功的實驗架構，並包含他們怎麼做到 Sim2Real。

[GitHub](https://github.com/awslabs/amazon-sagemaker-examples/tree/master/reinforcement_learning/rl_deepracer_robomaker_coach_gazebo)

## Introduction
要開發機器人的 RL 演算法需要很多方面的知識和工具，如
* access to a physical robot
* an accurate robot model for simulations
* a distributed training mechanism and customizability of the training procedure such as modifying the neural network and the loss function or  introducing noise.

DeepRacer 開發出一套可以讓 Simulation 和 Policy update 獨立運作的分散式計算架構，並支援 Gazebo。

DeepRacer 1/18th scale car 是該平台上使用的小車模型的實體版本，搭載用來做類神經網路運算的 GPU、透過 WiFi 傳輸的即時視訊鏡頭及 400 小時的續航力，成本為 400 鎂 (過去其他的開放小車硬體動輒上千鎂，這台因為沒有光達所以很便宜)

他們在這平台上，不依靠 expert labeling、real world，用 PPO 訓練了一個 end-to-end 的自動駕駛模型，訓練時間在 5 分鐘、約 5000 次模擬 (5000 次模擬在 5 分鐘內跑完 ... 又是一個用錢堆出來的研究)，且這個模型可以直接丟到實體的小車上使用。這是第一篇使用 model-free RL 做 sim2real 成功的研究。

他們的平台支援 sensor/policy log、在模擬環境中加入隨機性 (eg. random tracks, lighting, sensor/actuator noise)。

為了避免 overfit/underfit，他們開發了一套 robust evaluation method 去評估 policy 是否 generalize to real world，他們把同一個 policy 丟進多個 domain randomization 環境做 evaluation，並以此篩選出 robust policy，他們實驗證實在 domain randomization 中表現好的 policy 在 sim2real 的表現上也最好。

## Related Work

列舉了一些相關研究，有空再回來看

## Autonomous Racing with RL

在他們的架構裡面 agent 控制小車，environment 是賽車跑道，跑道上有白線表示跑道邊界，小車只會往前跑且路上沒有其他障礙，observation 是車子的鏡頭，action 是車子的油門/方向盤

![](https://i.imgur.com/g7umv0s.png)

因為車車不知道整個跑道長什麼樣子，因此這是個 partially observed Markov Decision Process。

一個 episode 車車從跑道某一點開始，如果車子跑出跑道或跑完一圈則結束。

鏡頭影像為 15 fps，resize 成 160 x 120 px 並轉成灰階。

Actions 被離散化成 10 個值，包含 2 level 的油門 * 5 level 的轉向 = 10 種 action，使用者可以在平台上自由調整該怎麼離散化。

訓練目標是讓車車越靠近中間線越好，因為如果車子太靠近邊邊的話，小小的改變就有可能讓跑道線超出視線範圍，使用者也可以自己定義 reward function。

### A. Reinforcement Learning Algorithm

他們使用 PPO，當時的 RL 領域的 SOTA，PPO 使用兩個 neural networks，policy network & value network。

Policy network input 是影像，output 是 action
Value network input 一樣是影像，output 用來估計期望的 discouted reward

初始 policy 是隨機的，每次跑完會根據蒐集到的資料更新 policy network 跟 value network，更新後的 policy 會再次被丟進 environment 跑模擬蒐集資料。

Policy loss function 利用 generalized advantage estimation algorithm 和 clipped importance sampling weight 最大化平均能夠給出最多 reward 的 action (詳情請見 [PPO 的論文](https://arxiv.org/pdf/1707.06347) 或這個簡短介紹的 [影片](https://www.youtube.com/watch?v=5P7I-xPq8u8))
Value loss function 使用 observed 跟 predicted reward 的 MSE

只有 policy network 需要被部屬到真實的車車上，他們兩個網路都使用 3 層 Conv + 2 層 FC，每 20 個 episode 更新一次權重。

## DeepRacer Design and Implementation

這章主要介紹他們如何將 policy update 和 simulation 分離。

他們使用 RoboMaker + Gazebo 做 simulation，使用 SageMaker + RL Coach 做 policy update。

把 simulation 和 policy update 分離的架構可以讓我們分別把這兩項工作交給為他們特化的機器來執行，例如 simulation 跑在 Windows 上 (不知道為什麼)，neural network training 跑在有 GPU 且 RAM 比較大的伺服器上。同時也可以使用多台機器平行的跑不同的 domain randomization 模擬。

### A. Training Workflow

![](https://i.imgur.com/BubQe3u.png)

上圖展示了整個訓練架構的流程。

訓練從在 SageMaker 中設定初始化 policy/value network 以及設定 hyper-parameter 開始，並利用 S3 儲存模型檔。

接下來 RobotMaker 會初始化模擬 environment、agent，並將 model 從 S3 下載下來，agent 透過 OpenAI Gym interface 和 environment 作互動，每一個 simulation step 中 agent 根據 observation $o$ 決定 action $a$，environment 會根據 $a$ 更新模擬環境並回傳下一個 observation 和當下的 reward，每一個 simulation step 的資料 $\langle o_t, a_t, r_t, o_{t+1} \rangle$ 都會透過 Redis 儲存起來。

接著 SageMaker 會根據 Redis 中的 replay data 訓練模型並將更新過後的模型再存進 S3，如此循環下去。

儲存在 S3 的模型會同時被丟進不同的環境中跑 simulation，蒐集更多資料，幫助收斂和 generalization。

這個架構不只可以訓練 PPO，也可以做 DQN、DDPG、SAC 等等常見的 RL 算法。

其中 RoboMaker 中的 environment 可以用其他和 OpenAi Gym interface 相容的 simulator 取代。

### B. Training with Amazon SageMaker

SageMaker 是 A 社基於 Jupyter Notebook 用來訓練 ML 模型的平台，並且可以用 RL Coach 和 RLlib 整合一些 RL 演算法。RL Coach 用在訓練和模擬分離的情境，RLlib 用在訓練和模擬整合的情境。

這些 library 都使用 Docker 打包，你可以把它們丟到不同設定的 cluster instance 上面跑，然後 AWS 就可以收錢錢。

SageMaker 除了支援車車訓練以外還有 HVAC、locomotion、資產管理等等實驗環境。

### C. Simulation with AWS RoboMaker

RoboMaker 是 A 社用來開發、測試和部屬機器人軟體的服務，使用 Gazebo 做 simulation。

`robot model` 是上面用來描述模型的資料結構 (類似 Gazebo 的 URDF)，例如定義車殼、車輪、Ackermann 轉向機構、相機，這些 component 的尺寸、物理參數、裝置參數、他們之間怎麼接合等等。

賽道跟背景是使用 Blender 製作的，做好之後再匯進 Gazebo 中。

物理引擎使用 ODE physics engine，渲染引擎使用 OGRE。

使用 Gazebo plugins 來增加 camera 和光源。

使用 ROS 作為 agent 和 simulation 的溝通介面。Agent 會透過 ROS 來擺放車車，並開始模擬、從相機模組取得影像、取得車車的位置、速度並將模型計算的 action (油門/轉向) 傳送給車車。

### D. Sim2Real Calibration

為了讓模擬更接近真實情況，他們作了以下幾個調整

首先相機部分的參數如高度、角度、FoV都盡量和真實的車子設定得一樣。

此外因為真實的相機只有 15 fps，他們將模擬環境的相機也調整成相同的 fps，並利用 producer-consumer 機制確保每個 frame 對應一個 action。

另外馬達控制部分，他們直接測量在不同的設定下的轉向角度跟速度來得到相關參數。

跑到部分也在現實世界中製作了和模擬環境相同配色、尺寸、形狀的跑道

欣賞一下他們製作的精美跑道
![](https://i.imgur.com/RSe7yPN.png)

### E. Calculating Rewards

這小節在介紹他們怎麼透過模型的幾何數據計算跑道中線和判斷車車是否超線的，不是很重要。

### F. DeepRacer Hardware

![](https://i.imgur.com/ztntBk9.png)

Intel Atom 搭載內置 GPU，使用剛剛提到 5 層的 neural networks 可達到 >15 inference per second。

使用 Intel OpenVINO 將 TensorFlow 的編譯成 binary 加速計算時間。

Camera 所取得到的影像會直接丟給 OpenVINO 做計算，同時透過 WiFi 在他們提供的 web interface 上做 live streaming，這個 web interface 也提供可以讓你手動操控車車的介面。

## Evaluation

為了讓 agent 能夠更好的 generalization，他們在模擬和真實的測試環境都加入不同的 domian randomization 來進行測試。

真實環境的部分，他們使用白色跟黃色膠帶製作跑道，分別在地毯或水泥地上、在周圍圍上檔板或沒有檔板、不同的光照條件、不同跑道形狀。

### A. Training with Multiple Rollouts

他們在以下三種情境下訓練 policy
1. Track A 最高時速 1 m/s
2. Track A 最高時速 1.67 m/s 
3. Track B 最高時速 1.67 m/s

依次對車車來說難度越來越高，Track B 難度高因為背景還有建築物之類的干擾。

每一個 episode 都會在跑道不同的位置重生，讓不同段的跑道都有機會被 agent 看到。

他們使用 p3.2x 來訓練，每次實驗都跑兩次，一次跑 2 個小時，實驗結果和預期相同，同時 rollout 的 worker 越多模型收斂越快，而且他們發現增加油門在 Track A 反而讓模型收斂更快，猜測是速度快讓 agent 能夠一次蒐集更多資料

![](https://i.imgur.com/5nBUQHm.png)

### B. Robust Evaluation

這部分是測試 robust evaluation (也就是在各種 domain randomization 下測試) 是否能夠有效的反應模型在真實世界的表現，如果可以的話，我們可以就可以直接在模擬環境下看 robust evaluation 來調整超參數而不用經常反覆的在現實環境中測試。

使用的 baseline 是 Track A w/ no domain randomization & 油門 1 m/s，實驗組為訓練時加入
(i). 在油門跟轉向加入至多 10% 的 uniform random noise
(ii). 轉換跑道方向
(iii). 同時使用 1. 和 2.
(iv). 同時使用 1. 和 2. 並且在 Track B 訓練

Robust evaluation 部分則是在測試的時後加入
1. uniform random noise to actions
2. in multiple starting position
3. both direction of travel

與其對照的是 naive evaluation，在固定的起始點且不做 randomization。

每個 checkpoint 都會對兩種測是各自執行 10 次，每 5 個 checkpoint 會取 6 個 policy 出來做 sim2real 的測試，sim2real 是用現實的車車在 Track A replica 上兩個方向各測試 3 次，模型表現與車子速度有關，但現實的車車售電量影響很難維持等速，且模型也會改變車子油門，最後他們想辦法盡量讓車子稍微比模擬快一點 (沒解釋為什麼)

實驗結果如下

![](https://i.imgur.com/JcjKh4I.png)

有以下結論
1. robust evaluation 表現好的 sim2real 表現也會比較好
2. reversing 對模型表現有很大的幫助
3. 只有 action noise 沒什麼幫助，但加入 reversing 之後看起來的幫助 (有嗎?)
4. Track B 一開始表現差，但隨著訓練越久表現越來越好

除此之外他們也發現速度低的話模型收斂速度更快而且很容易 generalization (這不是跟之前速度變快時收斂變快的想法相違背了嗎)，他們發現 0.5 m/s 時，checkpoint 5 中 baseline model 已經可以完成跑道。

### C. Robust Sim2Real

最後他們研究如何才能夠訓練出最 robust 的 agent，他們嘗試在訓練時調整以下因素
1. multiple track
2. multiple speeds
3. regularization
4. domain randomization in action
5. domain randomization in observation

他們先在 Track B 用 1 m/s 的油門，加入 action noise 和 reversing 訓練，然後根據 robust evaluation 挑選模型並在 Track A replica 加入以下情境測試其 sim2real 表現
1. two speeds (0.5 m/s, 1 m/s)
2. with bright sunlight
3. with no barriers
4. on tape track

以下結果
![](https://i.imgur.com/3AYzS0W.png)

以下結論
1. 在不同的跑道上訓練 sim2real 會比較好，但並不一定都比較好
2. Regularization 他們使用 L2 norm, dropout, BN, entropy bonus to policy loss，其中最顯著改善的是 entropy bonus 0.001, dropout w/ p=0.3.
3. 訓練時油門速度高有助於增加 robustness，但也會加長收斂的時間
4. 同時使用不同的跑道訓練沒什麼幫助
5. observation noise 他們使用 random color, horizontal translation, shadow, salt and pepper noise，其中 random color 是最有效的，random color 的作法是隨機指定 hue, saturation, brightness, contract 等

最後他們認為表現最好的是 Track C + L2 regularization + low entropy bouns + dropout + color randomization + 最高速度 2.33 m/s，可以在 11 秒內跑完 Track A 一圈

## Conclustion

總之就是 DeepRacer 架構設計很棒讓他們可以做各式各樣的測試，並且他們用 PPO 在蔗平台上訓練出了一個 end-to-end 基於影像的 navigation model，而且訓練速度很快。

## 小結

這篇就是在為自家服務打廣告，不過他們的架構設計和 sim2real 的技巧都是很好的參考。