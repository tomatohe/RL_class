# ROB6323 Go2 Project — Isaac Lab

This repository is the starter code for the NYU Reinforcement Learning and Optimal Control project in which students train a Unitree Go2 walking policy in Isaac Lab starting from a minimal baseline and improve it via reward shaping and robustness strategies. Please read this README fully before starting and follow the exact workflow and naming rules below to ensure your runs integrate correctly with the cluster scripts and grading pipeline.

## Repository policy

- Fork this repository and do not change the repository name in your fork.  
- Your fork must be named rob6323_go2_project so cluster scripts and paths work without modification.

### Prerequisites

- **GitHub Account:** You must have a GitHub account to fork this repository and manage your code. If you do not have one, [sign up here](https://github.com/join).

### Links
1.  **Project Webpage:** [https://machines-in-motion.github.io/RL_class_go2_project/](https://machines-in-motion.github.io/RL_class_go2_project/)
2.  **Project Tutorial:** [https://github.com/machines-in-motion/rob6323_go2_project/blob/master/tutorial/tutorial.md](https://github.com/machines-in-motion/rob6323_go2_project/blob/master/tutorial/tutorial.md)

## Connect to Greene

- Connect to the NYU Greene HPC via SSH; if you are off-campus or not on NYU Wi‑Fi, you must connect through the NYU VPN before SSHing to Greene.  
- The official instructions include example SSH config snippets and commands for greene.hpc.nyu.edu and dtn.hpc.nyu.edu as well as VPN and gateway options: https://sites.google.com/nyu.edu/nyu-hpc/accessing-hpc?authuser=0#h.7t97br4zzvip.

## Clone in $HOME

After logging into Greene, `cd` into your home directory (`cd $HOME`). You must clone your fork into `$HOME` only (not scratch or archive). This ensures subsequent scripts and paths resolve correctly on the cluster. Since this is a private repository, you need to authenticate with GitHub. You have two options:

### Option A: Via VS Code (Recommended)
The easiest way to avoid managing keys manually is to configure **VS Code Remote SSH**. If set up correctly, VS Code forwards your local credentials to the cluster.
- Follow the [NYU HPC VS Code guide](https://sites.google.com/nyu.edu/nyu-hpc/training-support/general-hpc-topics/vs-code) to set up the connection.

> **Tip:** Once connected to Greene in VS Code, you can clone directly without using the terminal:
> 1. **Sign in to GitHub:** Click the "Accounts" icon (user profile picture) in the bottom-left sidebar. If you aren't signed in, click **"Sign in with GitHub"** and follow the browser prompts to authorize VS Code.
> 2. **Clone the Repo:** Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`), type **Git: Clone**, and select it.
> 3. **Select Destination:** When prompted, select your home directory (`/home/<netid>/`) as the clone location.
>
> For more details, see the [VS Code Version Control Documentation](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git#_clone-a-repository-locally).

### Option B: Manual SSH Key Setup
If you prefer using a standard terminal, you must generate a unique SSH key on the Greene cluster and add it to your GitHub account:
1. **Generate a key:** Run the `ssh-keygen` command on Greene (follow the official [GitHub documentation on generating a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)).
2. **Add the key to GitHub:** Copy the output of your public key (e.g., `cat ~/.ssh/id_ed25519.pub`) and add it to your account settings (follow the [GitHub documentation on adding a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)).

### Execute the Clone
Once authenticated, run the following commands. Replace `<your-git-ssh-url>` with the SSH URL of your fork (e.g., `git@github.com:YOUR_USERNAME/rob6323_go2_project.git`).
```
cd $HOME
git clone <your-git-ssh-url> rob6323_go2_project
```
*Note: You must ensure the target directory is named exactly `rob6323_go2_project`. This ensures subsequent scripts and paths resolve correctly on the cluster.*
## Install environment

- Enter the project directory and run the installer to set up required dependencies and cluster-side tooling.  
```
cd $HOME/rob6323_go2_project
./install.sh
```
Do not skip this step, as it configures the environment expected by the training and evaluation scripts. It will launch a job in burst to set up things and clone the IsaacLab repo inside your greene storage. You must wait until the job in burst is complete before launching your first training. To check the progress of the job, you can run `ssh burst "squeue -u $USER"`, and the job should disappear from there once it's completed. It takes around **30 minutes** to complete. 
You should see something similar to the screenshot below (captured from Greene):

![Example burst squeue output](docs/img/burst_squeue_example.png)

In this output, the **ST** (state) column indicates the job status:
- `PD` = pending in the queue (waiting for resources).
- `CF` = instance is being configured.
- `R`  = job is running.

On burst, it is common for an instance to fail to configure; in that case, the provided scripts automatically relaunch the job when this happens, so you usually only need to wait until the job finishes successfully and no longer appears in `squeue`.

## What to edit

- In this project you'll only have to modify the two files below, which define the Isaac Lab task and its configuration (including PPO hyperparameters).  
  - source/rob6323_go2/rob6323_go2/tasks/direct/rob6323_go2/rob6323_go2_env.py  
  - source/rob6323_go2/rob6323_go2/tasks/direct/rob6323_go2/rob6323_go2_env_cfg.py
PPO hyperparameters are defined in source/rob6323_go2/rob6323_go2/tasks/direct/rob6323_go2/agents/rsl_rl_ppo_cfg.py, but you shouldn't need to modify them.

## How to edit

- Option A (recommended): Use VS Code Remote SSH from your laptop to edit files on Greene; follow the NYU HPC VS Code guide and connect to a compute node as instructed (VPN required off‑campus) (https://sites.google.com/nyu.edu/nyu-hpc/training-support/general-hpc-topics/vs-code). If you set it correctly, it makes the login process easier, among other things, e.g., cloning a private repo.
- Option B: Edit directly on Greene using a terminal editor such as nano.  
```
nano source/rob6323_go2/rob6323_go2/tasks/direct/rob6323_go2/rob6323_go2_env.py
```
- Option C: Develop locally on your machine, push to your fork, then pull changes on Greene within your $HOME/rob6323_go2_project clone.

> **Tip:** Don't forget to regularly push your work to github

## Launch training

- From $HOME/rob6323_go2_project on Greene, submit a training job via the provided script.  
```
cd "$HOME/rob6323_go2_project"
./train.sh
```
- Check job status with SLURM using squeue on the burst head node as shown below.  
```
ssh burst "squeue -u $USER"
```
Be aware that jobs can be canceled and requeued by the scheduler or underlying provider policies when higher-priority work preempts your resources, which is normal behavior on shared clusters using preemptible partitions.

## Where to find results

- When a job completes, logs are written under logs in your project clone on Greene in the structure logs/[job_id]/rsl_rl/go2_flat_direct/[date_time]/.  
- Inside each run directory you will find a TensorBoard events file (events.out.tfevents...), neural network checkpoints (model_[epoch].pt), YAML files with the exact PPO and environment parameters, and a rollout video under videos/play/ that showcases the trained policy.  

## Download logs to your computer

Use `rsync` to copy results from the cluster to your local machine. It is faster and can resume interrupted transfers. Run this on your machine (NOT on Greene):

```
rsync -avzP -e 'ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null' qf2128@dtn.hpc.nyu.edu:/home/qf2128/rob6323_go2_project/logs ./
```

*Explanation of flags:*
- `-a`: Archive mode (preserves permissions, times, and recursive).
- `-v`: Verbose output.
- `-z`: Compresses data during transfer (faster over network).
- `-P`: Shows progress bar and allows resuming partial transfers.

## Visualize with TensorBoard

You can inspect training metrics (reward curves, loss values, episode lengths) using TensorBoard. This requires installing it on your local machine.

1.  **Install TensorBoard:**
    On your local computer (do NOT run this on Greene), install the package:
    ```
    pip install tensorboard
    ```

2.  **Launch the Server:**
    Navigate to the folder where you downloaded your logs and start the server:
    ```
    # Assuming you are in the directory containing the 'logs' folder
    tensorboard --logdir ./logs
    ```

3.  **View Metrics:**
    Open your browser to the URL shown (usually `http://localhost:6006/`).

## Debugging on Burst

Burst storage is accessible only from a job running on burst, not from the burst login node. The provided scripts do not automatically synchronize error logs back to your home directory on Greene. However, you will need access to these logs to debug failed jobs. These error logs differ from the logs in the previous section.

The suggested way to inspect these logs is via the Open OnDemand web interface:

1.  Navigate to [https://ood-burst-001.hpc.nyu.edu](https://ood-burst-001.hpc.nyu.edu).
2.  Select **Files** > **Home Directory** from the top menu.
3.  You will see a list of files, including your `.err` log files.
4.  Click on any `.err` file to view its content directly in the browser.

> **Important:** Do not modify anything inside the `rob6323_go2_project` folder on burst storage. This directory is managed by the job scripts, and manual changes may cause synchronization issues or job failures.

## Project scope reminder

- The assignment expects you to go beyond velocity tracking by adding principled reward terms (posture stabilization, foot clearance, slip minimization, smooth actions, contact and collision penalties), robustness via domain randomization, and clear benchmarking metrics for evaluation as described in the course guidelines.  
- Keep your repository organized, document your changes in the README, and ensure your scripts are reproducible, as these factors are part of grading alongside policy quality and the short demo video deliverable.

## Resources

- [Isaac Lab documentation](https://isaac-sim.github.io/IsaacLab/main/source/setup/ecosystem.html) — Everything you need to know about IsaacLab, and more!
- [Isaac Lab ANYmal C environment](https://github.com/isaac-sim/IsaacLab/tree/main/source/isaaclab_tasks/isaaclab_tasks/direct/anymal_c) — This targets ANYmal C (not Unitree Go2), so use it as a reference and adapt robot config, assets, and reward to Go2.
- [DMO (IsaacGym) Go2 walking project page](https://machines-in-motion.github.io/DMO/) • [Go2 walking environment used by the authors](https://github.com/Jogima-cyber/IsaacGymEnvs/blob/e351da69e05e0433e746cef0537b50924fd9fdbf/isaacgymenvs/tasks/go2_terrain.py) • [Config file used by the authors](https://github.com/Jogima-cyber/IsaacGymEnvs/blob/e351da69e05e0433e746cef0537b50924fd9fdbf/isaacgymenvs/cfg/task/Go2Terrain.yaml) — Look at the function `compute_reward_CaT` (beware that some reward terms have a weight of 0 and thus are deactivated, check weights in the config file); this implementation includes strong reward shaping, domain randomization, and training disturbances for robust sim‑to‑real, but it is written for legacy IsaacGym and the challenge is to re-implement it in Isaac Lab.
- **API References**:
    - [ArticulationData (`robot.data`)](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.assets.html#isaaclab.assets.ArticulationData) — Contains `root_pos_w`, `joint_pos`, `projected_gravity_b`, etc.
    - [ContactSensorData (`_contact_sensor.data`)](https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.sensors.html#isaaclab.sensors.ContactSensorData) — Contains `net_forces_w` (contact forces).

---
Students should only edit README.md below this ligne.

# Student Modifications - qf2128


## Detailed Change Log

### 1. Action Rate Penalties (Smooth Motion Control)
The robot was making jerky movements. Adding smoothness penalties makes the walking more natural and stable.
**What Changed:**
- Added `last_actions` buffer (shape: `[num_envs, 12, 3]`) to track 3-step action history
- Implemented first derivative penalty: `||a_t - a_{t-1}||²`
- Implemented second derivative penalty: `||a_t - 2a_{t-1} + a_{t-2}||²`
- Added `action_rate_reward_scale = -0.01` in configuration

### 2. Custom Low-Level PD Controller
Isaac Lab's default controller was too basic. A custom controller gives us better torque control for more realistic robot behavior.

**What Changed:**
- Disabled implicit PD controller: `stiffness=0.0`, `damping=0.0`
- Implemented explicit torque calculation: `τ = Kp(q_des - q) - Kd·q̇`
- Added configurable gains: `Kp=20.0`, `Kd=0.5`, `torque_limits=100.0`
- Modified `_apply_action()` to use `set_joint_effort_target()` instead of `set_joint_position_target()`

### 3. Early Termination (Base Height Constraint)
Stopping training when the robot falls saves time. The 20cm height limit prevents the robot from learning to crawl instead of walk.

**What Changed:**
- Added `base_height_min = 0.20` (20cm threshold)
- Modified `_get_dones()` to check: `base_height < 0.20`

### 4. Raibert Heuristic (Intelligent Foot Placement)
This helps the robot place its feet in smart locations. It's based on real quadruped locomotion research and makes the gait more stable.

**What Changed:**
- Added gait phase tracking: `gait_indices`, `foot_indices`, `clock_inputs`
- Implemented trotting gait pattern (frequency=3Hz, phase offset=0.5)
- Implemented `_step_contact_targets()` for stance/swing phase calculation
- Implemented `_reward_raibert_heuristic()` for optimal foot position calculation
- Added 4 clock inputs to observation space (48 → 52 dimensions)

### 5. Stability Reward Shaping
The original task only cared about forward speed. These penalties keep the robot upright and reduce unwanted bouncing or rolling.

**What Changed:**
Added 4 new penalty terms:

1. **Orientation Penalty** (`orient_reward_scale = -1.0`)
   - Penalizes: `||projected_gravity_b[:2]||²`
   - Keeps robot upright (gravity should only project on Z-axis)

2. **Vertical Velocity Penalty** (`lin_vel_z_reward_scale = -0.002`)
   - Penalizes: `|v_z|²`
   - Reduces bouncing/hopping behavior

3. **Joint Velocity Penalty** (`dof_vel_reward_scale = -0.0001`)
   - Penalizes: `Σ||q̇||²`
   - Encourages smooth joint motion

4. **Angular Velocity XY Penalty** (`ang_vel_xy_reward_scale = -0.0001`)
   - Penalizes: `||ω_{xy}||²`
   - Reduces roll and pitch oscillations

### 6. Advanced Foot Interaction Rewards
Real dogs lift their feet during swing and press down during stance. These rewards teach the robot proper foot behavior for better walking.

**What Changed:**

1. **Foot Clearance Reward** (`feet_clearance_reward_scale = -3.0`)
   - During swing phase: penalizes feet below 5cm height
   - During stance phase: no penalty
   - Uses gait phase from `foot_indices` to determine swing/stance

2. **Contact Force Tracking Reward** (`tracking_contacts_shaped_force_reward_scale = 0.4`)
   - During stance phase: rewards contact force near 50N
   - During swing phase: penalizes any contact force
   - Uses exponential shaping: `exp(-|force_error| / 25.0)`

### Final Reward Composition
```bash
Total Reward = 
  # Tracking objectives
  + 2.0 * exp(-||v_xy_error||² / 0.25)           # Linear velocity
  + 0.5 * exp(-|ω_z_error|² / 0.25)              # Yaw rate
  
  # Smoothness
  - 0.01 * (||Δa||² + ||Δ²a||²)                  # Action rate
  
  # Gait shaping
  - 1.0 * ||foot_pos - raibert_target||²         # Raibert heuristic
  - 3.0 * Σ(clearance_error_during_swing)        # Foot clearance
  + 0.4 * Σ(exp(-|force_error| / 25))            # Contact forces
  
  # Stability
  - 1.0 * ||g_xy||²                              # Orientation
  - 0.002 * |v_z|²                               # Vertical velocity
  - 0.0001 * ||q̇||²                              # Joint velocities
  - 0.0001 * ||ω_xy||²                           # Angular velocity XY
```

## How to Reproduce Results

# Fork my repository: https://github.com/tomatohe/RL_class
```bash
# Then 
cd $HOME
git clone git@github.com:YOUR_USERNAME/rob6323_go2_project.git
```
### Training the Model
```bash
# Make sure you're in the project directory
cd $HOME/rob6323_go2_project

# Start training (this submits a job to the cluster)
./train.sh

# Check if your job is running
ssh burst "squeue -u $USER"
```

### Viewing Training Progress
```bash
# Download logs to your local machine (run this on your computer, not on Greene)
rsync -avzP -e 'ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null' YOUR_NETID@dtn.hpc.nyu.edu:/home/YOUR_NETID/rob6323_go2_project/logs ./

# Start TensorBoard (on your local machine)
cd logs
tensorboard --logdir .
# Then open http://localhost:6006 in your browser
```

### Key Metrics to Watch
- **rew_lin_vel**: Should increase (robot learning to walk forward)
- **rew_yaw_rate**: Should increase (robot learning to turn)
- **rew_feet_clearance**: Should become less negative (better foot lifting)
- **rew_tracking_contacts_shaped_force**: Should increase (better ground contact)
- **episode_length**: Should increase (robot surviving longer)

## Performance Results

### Training Results

The following TensorBoard screenshots show successful training with my modifications:

#### Reward Components Progress
![Episode Rewards](figure/Screenshot%20from%202025-12-18%2019-07-20.png)
*All reward components show steady improvement: Raibert heuristic, action rate penalties, foot clearance, contact forces, and tracking rewards.*

#### Training Stability Metrics  
![Training Stability](figure/Screenshot%20from%202025-12-18%2019-09-20.png)
*Episode termination decreases and loss functions converge properly, showing stable training.*

#### Performance Metrics
![Performance Stats](figure/Screenshot%20from%202025-12-18%2019-09-27.png)
*Consistent collection time (~1.3s), stable learning time, and good FPS performance.*

#### Overall Training Progress
![Training Summary](figure/Screenshot%20from%202025-12-18%2019-09-35.png)
*Episode length increases to ~1000 steps, episode rewards reach ~3000, showing successful learning.*

### Key Achievements
- **Episode Length**: Increased from ~300 to 1000+ steps (robot survives longer)
- **Total Reward**: Reached ~3000 (strong positive performance)
- **Stability**: Low termination rate after initial training phase
- **Foot Clearance**: Reward improved from -6 to -3 (better foot lifting)
- **Contact Forces**: Positive reward ~40+ (proper ground contact)
- **Smooth Actions**: Action rate penalties successfully reduced jerkiness

