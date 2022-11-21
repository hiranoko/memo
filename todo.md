
## RL

- データ不均衡の対応

サンプリングの頻度を調整する

- 単一ネットワークで複数エージェントへの指令

[Code](https://github.com/kuto5046/kaggle-luxai/tree/main/exp/exp054)

[Dataset](https://www.kaggle.com/datasets/kuto0633/lux-ai-top-episodes)

- LeanerとTeacherが乖離しないように監視するための損失関数

```python
def compute_teacher_kl_loss(
        learner_policy_logits: torch.Tensor,
        teacher_policy_logits: torch.Tensor,
        actions_taken_mask: torch.Tensor
) -> torch.Tensor:
    learner_policy_log_probs = F.log_softmax(learner_policy_logits, dim=-1)
    teacher_policy = F.softmax(teacher_policy_logits, dim=-1)
    kl_div = F.kl_div(
        learner_policy_log_probs,
        teacher_policy.detach(),
        reduction="none",
        log_target=False
    ).sum(dim=-1)
    assert actions_taken_mask.shape == kl_div.shape
    kl_div_masked = kl_div * actions_taken_mask.float()
    # Sum over y, x, and action_planes dimensions to combine kl divergences from different actions
    return kl_div_masked.sum(dim=-1).sum(dim=-1).squeeze(dim=-2)
```

### LuxAI

- 1st  
https://github.com/IsaiahPressman/Kaggle_Lux_AI_2021

- 5th  
https://www.kaggle.com/competitions/lux-ai-2021/discussion/293911

- 6th  
https://www.kaggle.com/competitions/lux-ai-2021/discussion/293776

- 34th  
https://github.com/kuto5046/kaggle-luxai  
https://www.kaggle.com/c/lux-ai-2021/discussion/294003  
https://kutohonn.hatenablog.com/entry/2021/12/14/100303
