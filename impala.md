# Title

IMPALA: Scalable Distributed Deep-RL with Importance Weighted Actor-Learner Architectures

## Author

Lasse Espeholt

## Abstruct

- V-traceに非同期実行を加えた分散強化学習

## Memo

- Actor-Criticの設定を用いて方策 $\pi$ と価値関数 $V^\pi$ を学習する．

- Actorは $n$ ステップごとにデータ収集を行います．まず自身の方策 $\mu$ をLearnerの方策 $\pi$ に更新し，$n$ ステップ間データ収集を行います．その後，収集した経験データ(状態，行動，報酬)，方策の分布 $\mu(a_t|x_t)$ ，LSTMの初期状態をLearnerに送ります．一方，Learnerは複数のActorから送られてきたデータを用いて，繰り返し学習を行います．

- データ収集時の方策 $\mu$ は挙動方策、学習している方策 $\pi$ は推定方策と呼ばれる。方策の違いはV-traceで補正している。

- 学習時に発散させないため，$\rho$と$c$のClippingを導入している

$$\rho_i = \min(\bar \rho,\frac{\pi(a_i|x_i)}{\mu(a_i|x_i)})$$

$$c_i = \min(\bar c, \frac{\pi(a_i|x_i)}{\mu(a_i|x_i)})$$

### [V-trace](https://qiita.com/ku2482/items/c25c33aeae293434912d)

**V-traceオペレータ**

$n$ ステップの *V-trace* オペレータ $\mathcal R^n$ を以下のように定義します．

$$
\mathcal R^n V(x_s) := V(x_s) + \mathbb E_\mu[
\sum_{t=s}^{s+n-1} \gamma^{t-s}(c_s\cdots c_{t-1})\delta_tV
]
$$

ここで，$\delta_tV := \rho_{t}(r_t+ \gamma V(x_{t+1})-V(x_t))$ は，*Importance Sampling* (IS)により重み付けられたTD誤差を表します．また，$\rho_i = \min(\bar \rho,\frac{\pi(a_i|x_i)}{\mu(a_i|x_i)})$ ，$c_i = \min(\bar c, \frac{\pi(a_i|x_i)}{\mu(a_i|x_i)})$ は，クリップされたISの重み係数を表し，クリップの閾値は $\bar \rho \ge \bar c$ を満たすこととします．

ここで，*on-policy* ( $\mu = \pi$ ) の場合には，

$$
\begin{align}
\mathcal R^n V(x_s) &= V(x_s) + \mathbb E_\mu[
\sum_{t=s}^{s+n-1} \gamma^{t-s} (r_t+ \gamma V(x_{t+1})-V(x_t))] \\
&= \mathbb E_\mu[
\sum_{t=s}^{s+n-1}\gamma^{t-s} r_t + \gamma^n V(x_{s+n})]
\end{align}
$$

と変形でき，***on-policy* の場合の *V-trace* は，オンライン学習時の $n$ ステップベルマンオペレータに一致する**ことがわかります．

*V-trace* において，2つの異なるIS重み係数の閾値 $\bar \rho$ と $\bar c$ は異なる役割を果たします．

まず **重み係数 $\rho_i$ の閾値 $\bar \rho$ は，*V-trace* オペレータの唯一の不動点を定義している** と考えることができます．関数近似誤差を生じない表形式の場合には，*V-trace* オペレータは，以下の式で表される方策 $\pi^{\bar \rho}$ の価値関数 $V^{\pi_{\bar \rho}}$ を唯一の不動点として持ちます(実際に計算すると，確認できます)．

$$
\pi_{\bar \rho}(a|x) := \frac{\min(\bar \rho \mu(a|x), \pi(a|x))}{\sum_b\min(\bar \rho \mu(b|x), \pi(b|x))}
$$

よって，$\bar \rho$ が無限のとき，*V-trace* オペレータは方策 $\pi$ の価値関数 $V^\pi$ を唯一の不動点として持ち．$\bar \rho < \infty$ の場合には，$\pi$ と $\mu$ の間の方策の価値関数を不動点として持つことになります．**ISを行うと，行動方策 $\mu$ の確率密度が低いデータにおいて重み係数が非常に大きくなり，推定分散が大きくなってしまう**ので，$\bar \rho$ でクリップすることで分散を抑制します．よって，$\bar \rho$ が大きいときほど *off-policy* の学習における分散が大きく(一方でバイアスが小さく)，$\bar \rho$ が小さくなるにつれ分散が小さく(バイアスが大きく)なります．また，$\rho_i$ は $c_i$ と異なり時系列で掛け合わせる操作を行わないので，時系列によって発散するような挙動は生じません．

次に **重み係数 $c_i$ の閾値 $\bar c$ は，*V-trace* オペレータの収束の速さを制御している** と考えることができます．重み係数の掛け合わせ $(c_s\cdots c_{t-1})$ は，時刻 $t$ でのTD誤差 $\delta_t V =\rho_t (r_t + \gamma V(x_{t+1}) - V(x_t))$ が時刻 $s$ の価値関数 $V(x_s)$ の更新にどの程度寄与するかを評価しています．$c_i$ は時系列で掛け合わせる操作を伴うため発散しやすく，分散抑制のために重み係数 $c_i$ をクリップすることが重要です．ここで，**$\bar c$ の大きさは *V-trace* オペレータの不動点(学習が収束する点)には影響を与えない** ので，より分散を抑制するために $\bar \rho$ よりも小さな値に設定することが望ましいです．

実際には，*V-trace* は以下のように再帰的に計算することができます．

$$
\mathcal R^n V(x_t) = V(x_t) + + \mathbb E_{\mu}[
\delta_t V+ \gamma c_t (\mathcal R^n V(x_{t+1}) - V(x_{t+1}))
]
$$

**V-trace Actor-Critic**

方策 $\pi_\omega$ をパラメータ $\omega$ で，価値関数 $V_\theta$ をパラメータ $\theta$ で関数近似します．経験データは行動方策 $\mu$ で収集されたものとします．

価値関数の学習には，*V-trace* オペレータにおけるTD誤差を損失関数として用います．

$$
L_\theta = (\mathcal R^n V(x_s) - V_\theta(x_s))^2
$$

勾配は容易に計算可能で，以下の式で表せます．

$$
\nabla_\theta L_\theta = (\mathcal R^n V(x_s) - V_\theta(x_s)) \nabla_\theta V_\theta(x_s)
$$

また，方策勾配定理とISにより，方策 $\pi_{\bar\rho}$ の勾配は以下の式で表せます．

$$
E_{a_s\sim \mu}[\frac{\pi_{\bar \rho}(a_s|x_s)}{\mu(a_s|x_s)} \nabla_\omega \log \pi_{\bar \rho}(a_s|x_s) (q_s - b(x_s)) | x_s]
$$

ここで，$q_s = r_s + \gamma \mathcal R^n V(x_{s+1})$ は *V-trace* オペレータの下での行動価値関数 $Q^{\pi_\omega}(x_s,a_s)$ の推定値を表し，$b(x_s)$ は分散を抑制するための状態依存のベースライン関数を表します．クリッピングよるバイアスが極めて小さい( $\bar \rho$ が十分大きい)場合，上述の勾配は $\pi_{\omega}$ の方策勾配の良い推定値であると考えられます．よって，ベースライン関数に$V_\theta(x_s)$ を用いることで，以下の方策勾配を得ます．

$$
\nabla_\omega L_\omega = \rho_s \nabla_\omega \log \pi_\omega(a_s|x_s) (
r_s + \gamma \mathcal R^n V(x_{s+1}) - V_\theta(x_s))
$$

また，方策 $\pi_\omega$ が局所解に収束してしまうのを防ぐため，エントロピー損失を加えることも考えられます．

$$
L_{\rm ent} = - \sum_a \pi_\omega(a|x_s)\log \pi_\omega(a|x_s)
$$

### [torch.distributions](https://pytorch.org/docs/stable/distributions.html)

確率密度関数がパラメータを持ち、微分可能であるときにREINFORCEがpytorchで実装できる．実装コードは以下の通り、

```python
probs = policy_network(state)
m = Categorical(probs)
action = m.sample()
next_state, reward = env.step(action)
loss = -m.log_prob(action) * reward
loss.backward()
```

$$
\Delta \theta = \alpha r \frac{\partial \log p(a | \pi^\theta (s))}{\partial \theta}
$$

ここで、$\theta$はパタメータ，$\alpha$は学習率，$r$は報酬，$p(a | \pi^\theta (s))$は行動$a$を取り出す確率を表す。

### [minimalRL](https://github.com/seungeunrho/minimalRL)

**V-trace**

```python
import gym
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torch.distributions import Categorical
import numpy as np

#Hyperparameters
learning_rate      = 0.0005
gamma              = 0.98
T_horizon          = 20
clip_rho_threshold = 1.0
clip_c_threshold   = 1.0
print_interval     = 20

class Vtrace(nn.Module):
    def __init__(self):
        super(Vtrace, self).__init__()
        self.data = []
        
        self.fc1   = nn.Linear(4,256)
        self.fc_pi = nn.Linear(256,2)
        self.fc_v  = nn.Linear(256,1)
        self.optimizer = optim.Adam(self.parameters(), lr=learning_rate)

        self.clip_rho_threshold = torch.tensor(clip_rho_threshold, dtype=torch.float)
        self.clip_c_threshold = torch.tensor(clip_c_threshold, dtype=torch.float)

    def pi(self, x, softmax_dim = 0):
        x = F.relu(self.fc1(x))
        x = self.fc_pi(x)
        prob = F.softmax(x, dim=softmax_dim)
        return prob
    
    def v(self, x):
        x = F.relu(self.fc1(x))
        v = self.fc_v(x)
        return v
      
    def put_data(self, transition):
        self.data.append(transition)
        
    def make_batch(self):
        s_lst, a_lst, r_lst, s_prime_lst, mu_a_lst, done_lst = [], [], [], [], [], []
        for transition in self.data:
            s, a, r, s_prime, mu_a, done = transition
            
            s_lst.append(s)
            a_lst.append([a])
            r_lst.append([r])
            s_prime_lst.append(s_prime)
            mu_a_lst.append([mu_a])
            done_mask = 0 if done else 1
            done_lst.append([done_mask])
            
        s,a,r,s_prime,done_mask, mu_a = torch.tensor(s_lst, dtype=torch.float), torch.tensor(a_lst), \
                                        torch.tensor(r_lst), torch.tensor(s_prime_lst, dtype=torch.float), \
                                        torch.tensor(done_lst, dtype=torch.float), torch.tensor(mu_a_lst)
        self.data = []
        return s, a, r, s_prime, done_mask, mu_a

    def vtrace(self, s, a, r, s_prime, done_mask, mu_a):
        with torch.no_grad():
            pi = self.pi(s, softmax_dim=1)
            pi_a = pi.gather(1,a)
            v, v_prime = self.v(s), self.v(s_prime)
            ratio = torch.exp(torch.log(pi_a) - torch.log(mu_a))  # a/b == exp(log(a)-log(b))
            
            rhos = torch.min(self.clip_rho_threshold, ratio)
            cs = torch.min(self.clip_c_threshold, ratio).numpy()
            td_target = r + gamma * v_prime * done_mask
            delta = rhos*(td_target - v).numpy()
            
            vs_minus_v_xs_lst = []
            vs_minus_v_xs = 0.0
            vs_minus_v_xs_lst.append([vs_minus_v_xs])
            
            for i in range(len(delta)-1, -1, -1):
                vs_minus_v_xs = gamma * cs[i][0] * vs_minus_v_xs + delta[i][0]
                vs_minus_v_xs_lst.append([vs_minus_v_xs])
            vs_minus_v_xs_lst.reverse()
            
            vs_minus_v_xs = torch.tensor(vs_minus_v_xs_lst, dtype=torch.float)
            vs = vs_minus_v_xs[:-1] + v.numpy()
            vs_prime = vs_minus_v_xs[1:] + v_prime.numpy()
            advantage = r + gamma * vs_prime - v.numpy()
            
        return vs, advantage, rhos

    def train_net(self):
        s, a, r, s_prime, done_mask, mu_a = self.make_batch()
        vs, advantage, rhos = self.vtrace(s, a, r, s_prime, done_mask, mu_a)

        pi = self.pi(s, softmax_dim=1)
        pi_a = pi.gather(1,a)
       
        val_loss = F.smooth_l1_loss(self.v(s) , vs)
        pi_loss = -rhos * torch.log(pi_a) * advantage
        loss =  pi_loss + val_loss

        self.optimizer.zero_grad()
        loss.mean().backward()
        self.optimizer.step()
        
def main():
    env = gym.make('CartPole-v1')
    model = Vtrace()
    score = 0.0
    
    for n_epi in range(10000):
        s = env.reset()
        done = False
        while not done:
            for t in range(T_horizon):
                prob = model.pi(torch.from_numpy(s).float())
                m = Categorical(prob)
                a = m.sample().item()
                s_prime, r, done, info = env.step(a)

                model.put_data((s, a, r/100.0, s_prime, prob[a].item(), done))
                s = s_prime

                score += r
                if done:
                    break

            model.train_net()

        if n_epi%print_interval==0 and n_epi!=0:
            print("# of episode :{}, avg score : {:.1f}".format(n_epi, score/print_interval))
            score = 0.0

    env.close()

if __name__ == '__main__':
    main()
```