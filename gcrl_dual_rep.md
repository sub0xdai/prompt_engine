```json
{
  "schema_version": "gcrl_dual_representation_pipeline_1.2_template",
  "task_id": "<UNIQUE_TASK_ID>",
  "task_description": "A generic prompt template to train a GCRL agent using the 'Dual Goal Representation' methodology (Park et al., 2025).",

  "parameters": {
    "task_name": "<USER_DEFINED_TASK_NAME>",
    "description": "<USER_DEFINED_TASK_DESCRIPTION>",
    "offline_dataset_path": "<PATH_TO_OFFLINE_DATASET>",
    
    "state_definition": {
      "type": "<'image'|'vector'|...>",
      "format": "<'64x64x3'|'[768]'|...>"
    },
    "action_definition": {
      "type": "<'continuous'|'discrete'>",
      "dimensions": "<INTEGER_DIMENSION_COUNT>"
    },
    "raw_goal_definition": {
      "description": "<DESCRIPTION_OF_RAW_GOAL_FORMAT>",
      "type": "<'image'|'vector'|...>",
      "format": "<'64x64x3'|'[768]'|...>"
    },
    
    "representation_algorithm": {
      "name": "<e.g., 'GoalConditioned_IQL'>",
      "config": {
        "representation_dimensionality": "<e.g., 256>",
        "reward_function": "<e.g., 'sparse_negative_one_zero'>",
        "expectile_kappa": "<e.g., 0.7>",
        "...": "any other hyperparameters for the representation algorithm"
      }
    },
    
    "policy_algorithm": {
      "name": "<e.g., 'GCIVL'|'CRL'|'GCFBC'>",
      "config": {
        "...": "any hyperparameters for the downstream policy algorithm"
      }
    },
    
    "output_paths": {
      "goal_encoder": "<PATH_TO_SAVE_ENCODER_MODEL>",
      "final_policy": "<PATH_TO_SAVE_FINAL_POLICY>"
    }
  },

  "execution_plan": {
    "description": "This plan implements the 'Dual Goal Representation' methodology. It separates goal representation learning from policy learning.",
    
    "phase_1_learn_goal_representation": {
      "objective": "Learn a 'dual goal representation' phi(g) that is sufficient for control and invariant to exogenous noise.",
      "method": "Approximate the temporal distance function V*(s, g) using an inner product parameterization: V*(s,g) ≈ psi(s)ᵀ * phi(g).",
      "algorithm": "from:parameters.representation_algorithm.name",
      "inputs": {
        "state_data": "from:parameters.state_definition",
        "raw_goal_data": "from:parameters.raw_goal_definition",
        "dataset": "from:parameters.offline_dataset_path"
      },
      "config": "from:parameters.representation_algorithm.config",
      "outputs": {
        "goal_encoder_artifact_name": "from:parameters.output_paths.goal_encoder"
      }
    },

    "phase_2_learn_policy": {
      "objective": "Learn the final goal-conditioned policy pi(a | s, phi(g)) using the learned, frozen goal representation.",
      "dependencies": [
        "phase_1_learn_goal_representation"
      ],
      "algorithm": "from:parameters.policy_algorithm.name",
      "inputs": {
        "state_data": "from:parameters.state_definition",
        "learned_goal_representation": "from:phase_1_learn_goal_representation.outputs.goal_encoder_artifact_name",
        "dataset": "from:parameters.offline_dataset_path"
      },
      "config": "from:parameters.policy_algorithm.config",
      "outputs": {
        "final_policy_artifact_name": "from:parameters.output_paths.final_policy"
      }
    }
  }
}
```
