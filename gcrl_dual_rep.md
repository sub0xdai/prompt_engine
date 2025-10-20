```json
{
  "schema_version": "gcrl_dual_representation_pipeline_1.0",
  "task_id": "USER_TASK_001",
  "task_description": "Train an agent to perform a specific goal-conditioned task by first learning a noise-invariant goal representation.",

  "user_inputs": {
    "task_name": "Example: Robotic Arm Cube Stacking",
    "description": "Train a 7-DoF arm to pick up cube 'A' and place it on top of cube 'B'.",
    "offline_dataset_path": "/path/to/trajectories.dataset",
    "state_definition": {
      "type": "image",
      "format": "64x64x3_pixels"
    },
    "action_definition": {
      "type": "continuous",
      "dimensions": 7
    },
    "raw_goal_definition": {
      "description": "A raw image observation of the desired final scene.",
      "type": "image",
      "format": "64x64x3_pixels"
    }
  },

  "execution_plan": {
    "description": "This plan implements the 'Dual Goal Representation' methodology (Park et al., 2025). It separates goal representation learning from policy learning.",
    
    "phase_1_learn_goal_representation": {
      "objective": "Learn a 'dual goal representation' phi(g) that is sufficient for control and invariant to exogenous noise (e.g., lighting, background).",
      "method": "Approximate the temporal distance function V*(s, g) using an inner product parameterization: V*(s,g) ≈ psi(s)ᵀ * phi(g).",
      "algorithm": "GoalConditioned_IQL",
      "inputs": {
        "state_data": "from:user_inputs.state_definition",
        "raw_goal_data": "from:user_inputs.raw_goal_definition",
        "dataset": "from:user_inputs.offline_dataset_path"
      },
      "config": {
        "reward_function": "sparse_negative_one_zero",
        "expectile_kappa": 0.7,
        "representation_dimensionality": 256
      },
      "outputs": {
        "goal_encoder_artifact_name": "phi_encoder.model"
      }
    },

    "phase_2_learn_policy": {
      "objective": "Learn the final goal-conditioned policy pi(a | s, phi(g)) using the learned, frozen goal representation.",
      "dependencies": [
        "phase_1_learn_goal_representation"
      ],
      "algorithm": "GCIVL",
      "inputs": {
        "state_data": "from:user_inputs.state_definition",
        "learned_goal_representation": "from:phase_1_learn_goal_representation.outputs.goal_encoder_artifact_name",
        "dataset": "from:user_inputs.offline_dataset_path"
      },
      "config": {
        "policy_input_signature": "pi(action | state, learned_goal_representation)",
        "expectile_kappa": 0.9
      },
      "outputs": {
        "final_policy_artifact_name": "final_policy.model"
      }
    }
  }
}
```
