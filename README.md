# bone-snapping-animate
class MuscleNode:
    def __init__(self, name):
        self.name = name
        self.tension = 0.5        # abstract “load” state, 0-1
        self.load = 0.0
        self.max_capacity = 1.0
        self.response_speed = 1.0  # extraordinarily fast changes

    def receive_signal(self, intensity):
        # Abstract “sensory input” into the node
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.tension += delta
        self.tension = min(1.0, max(0.0, self.tension))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        # Abstract decay per tick
        self.load *= 0.9 ** (1.0 + self.response_speed)

class MuscleSystemSandbox:
    def __init__(self, muscle_names):
        self.muscles = [MuscleNode(name) for name in muscle_names]
        self.time = 0
        self.history = []

    def step(self, signal_intensity=1.0):
        self.time += 1
        step_data = {"time": self.time}
        for muscle in self.muscles:
            leftover = muscle.receive_signal(signal_intensity)
            muscle.step()
            step_data[f"{muscle.name}_tension"] = muscle.tension
            step_data[f"{muscle.name}_load"] = muscle.load
            step_data[f"{muscle.name}_leftover"] = leftover
        self.history.append(step_data)
import matplotlib.pyplot as plt
import random

# ------------------ Skeletal Node ------------------
class BoneNode:
    def __init__(self, name):
        self.name = name
        self.integrity = 0.5       # symbolic state of the bone/node
        self.load = 0.0             # abstract load applied
        self.max_capacity = 1.0
        self.response_speed = 1.0  # extraordinarily fast

    def receive_signal(self, intensity):
        # Apply signal with maximal numeric effect
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.integrity += delta
        self.integrity = min(1.0, max(0.0, self.integrity))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        # Abstract decay of load
        self.load *= 0.9 ** (1.0 + self.response_speed)

# ------------------ Skeletal Sandbox ------------------
class SkeletalSandbox:
    def __init__(self, bone_names):
        self.bones = [BoneNode(name) for name in bone_names]
        self.time = 0
        self.history = []

    def step(self, signal_intensity=1.0):
        self.time += 1
        step_data = {"time": self.time}
        for bone in self.bones:
            leftover = bone.receive_signal(signal_intensity)
            bone.step()
            step_data[f"{bone.name}_integrity"] = bone.integrity
            step_data[f"{bone.name}_load"] = bone.load
            step_data[f"{bone.name}_leftover"] = leftover
        self.history.append(step_data)

# ------------------ Define All Bones ------------------
bone_names = [
    # Skull
    "Frontal", "Parietal_L", "Parietal_R", "Occipital", "Temporal_L", "Temporal_R",
    "Sphenoid", "Ethmoid", "Maxilla_L", "Maxilla_R", "Mandible",
    # Spine
    "C1", "C2", "C3", "C4", "C5", "C6", "C7",
    "T1", "T2", "T3", "T4", "T5", "T6", "T7", "T8", "T9", "T10", "T11", "T12",
    "L1", "L2", "L3", "L4", "L5",
    "Sacrum", "Coccyx",
    # Thorax
    "Sternum", "Rib1_L", "Rib1_R", "Rib2_L", "Rib2_R", "Rib3_L", "Rib3_R",
    "Rib4_L", "Rib4_R", "Rib5_L", "Rib5_R", "Rib6_L", "Rib6_R", "Rib7_L", "Rib7_R",
    "Rib8_L", "Rib8_R", "Rib9_L", "Rib9_R", "Rib10_L", "Rib10_R", "Rib11_L", "Rib11_R",
    "Rib12_L", "Rib12_R",
    # Upper Limbs
    "Clavicle_L", "Clavicle_R", "Scapula_L", "Scapula_R", "Humerus_L", "Humerus_R",
    "Radius_L", "Radius_R", "Ulna_L", "Ulna_R", "Carpals_L", "Carpals_R",
    "Metacarpals_L", "Metacarpals_R", "Phalanges_L", "Phalanges_R",
    # Lower Limbs
    "Pelvis_L", "Pelvis_R", "Femur_L", "Femur_R", "Patella_L", "Patella_R",
    "Tibia_L", "Tibia_R", "Fibula_L", "Fibula_R", "Tarsals_L", "Tarsals_R",
    "Metatarsals_L", "Metatarsals_R", "PhalangesFoot_L", "PhalangesFoot_R"
]

# ------------------ Initialize Sandbox ------------------
skeletal_sandbox = SkeletalSandbox(bone_names)

# ------------------ Run World-Built Skeletal System ------------------
for _ in range(15):
    skeletal_sandbox.step(signal_intensity=1.0)  # maximal symbolic input

# ------------------ Visualize ------------------
plt.figure(figsize=(16, 8))
for bone in skeletal_sandbox.bones:
    plt.plot(
        [h["time"] for h in skeletal_sandbox.history],
        [h[f"{bone.name}_integrity"] for h in skeletal_sandbox.history],
        label=f"{bone.name} Integrity"
    )

plt.title("World-Built Skeletal System Sandbox: Maximal Signal Propagation")
plt.xlabel("Time Step")
plt.ylabel("Abstract Integrity (0-1)")
plt.grid(True)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')
plt.tight_layout()
plt.show()
# Internally
BoneNode("Femur_L")  # technically a node

# For output/visualization
label = "Femur Left"  # user-facing name
class LabeledNode:
    def __init__(self, name, type_label, category):
        self.name = name          # user-facing name
        self.type_label = type_label  # e.g., "bone"
        self.category = category      # e.g., noun, structural part
        self.state = 0.5
        self.load = 0.0
        self.response_speed = 1.0
import matplotlib.pyplot as plt

# ------------------ Labeled Bone Node ------------------
class BoneNode:
    def __init__(self, name):
        self.name = name           # User-facing bone name
        self.type_label = "bone"   # Classification: bone
        self.category = "noun"     # World-building category
        self.integrity = 0.5       # Abstract numeric state
        self.load = 0.0
        self.max_capacity = 1.0
        self.response_speed = 1.0  # Level 10 by default

    def receive_signal(self, intensity):
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.integrity += delta
        self.integrity = min(1.0, max(0.0, self.integrity))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        # Abstract decay
        self.load *= 0.9 ** (1.0 + self.response_speed)

# ------------------ Skeletal Sandbox ------------------
class SkeletalSandbox:
    def __init__(self, bone_names):
        self.bones = [BoneNode(name) for name in bone_names]
        self.time = 0
        self.history = []

    def step(self, signal_intensity=1.0):
        self.time += 1
        step_data = {"time": self.time}
        for bone in self.bones:
            leftover = bone.receive_signal(signal_intensity)
            bone.step()
            step_data[f"{bone.name}_integrity"] = bone.integrity
            step_data[f"{bone.name}_load"] = bone.load
            step_data[f"{bone.name}_leftover"] = leftover
        self.history.append(step_data)

# ------------------ Define All Bones ------------------
bone_names = [
    # Skull
    "Frontal", "Parietal_L", "Parietal_R", "Occipital", "Temporal_L", "Temporal_R",
    "Sphenoid", "Ethmoid", "Maxilla_L", "Maxilla_R", "Mandible",
    # Spine
    "C1", "C2", "C3", "C4", "C5", "C6", "C7",
    "T1", "T2", "T3", "T4", "T5", "T6", "T7", "T8", "T9", "T10", "T11", "T12",
    "L1", "L2", "L3", "L4", "L5",
    "Sacrum", "Coccyx",
    # Thorax
    "Sternum", "Rib1_L", "Rib1_R", "Rib2_L", "Rib2_R", "Rib3_L", "Rib3_R",
    "Rib4_L", "Rib4_R", "Rib5_L", "Rib5_R", "Rib6_L", "Rib6_R", "Rib7_L", "Rib7_R",
    "Rib8_L", "Rib8_R", "Rib9_L", "Rib9_R", "Rib10_L", "Rib10_R", "Rib11_L", "Rib11_R",
    "Rib12_L", "Rib12_R",
    # Upper Limbs
    "Clavicle_L", "Clavicle_R", "Scapula_L", "Scapula_R", "Humerus_L", "Humerus_R",
    "Radius_L", "Radius_R", "Ulna_L", "Ulna_R", "Carpals_L", "Carpals_R",
    "Metacarpals_L", "Metacarpals_R", "Phalanges_L", "Phalanges_R",
    # Lower Limbs
    "Pelvis_L", "Pelvis_R", "Femur_L", "Femur_R", "Patella_L", "Patella_R",
    "Tibia_L", "Tibia_R", "Fibula_L", "Fibula_R", "Tarsals_L", "Tarsals_R",
    "Metatarsals_L", "Metatarsals_R", "PhalangesFoot_L", "PhalangesFoot_R"
]

# ------------------ Initialize Sandbox ------------------
skeletal_sandbox = SkeletalSandbox(bone_names)

# ------------------ Apply Level 10 to All Bones ------------------
for bone in skeletal_sandbox.bones:
    bone.response_speed = 1.0  # maximum speed

# ------------------ Run Sandbox ------------------
for _ in range(15):
    skeletal_sandbox.step(signal_intensity=1.0)  # maximal input

# ------------------ Visualize ------------------
plt.figure(figsize=(16, 8))
for bone in skeletal_sandbox.bones:
    plt.plot(
        [h["time"] for h in skeletal_sandbox.history],
        [h[f"{bone.name}_integrity"] for h in skeletal_sandbox.history],
        label=f"{bone.name} ({bone.type_label})"
    )

plt.title("World-Built Skeletal Sandbox: Level 10 Signal Propagation")
plt.xlabel("Time Step")
plt.ylabel("Abstract Integrity (0-1)")
plt.grid(True)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')
plt.tight_layout()
plt.show()
# Update BoneNode class to include density
class BoneNode:
    def __init__(self, name):
        self.name = name
        self.type_label = "bone"
        self.category = "noun"
        self.integrity = 0.5
        self.load = 0.0
        self.density = 0.10        # new property
        self.max_capacity = 1.0
        self.response_speed = 1.0

    def receive_signal(self, intensity):
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.integrity += delta
        self.integrity = min(1.0, max(0.0, self.integrity))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        self.load *= 0.9 ** (1.0 + self.response_speed)
# Updated BoneNode class with density and strength
class BoneNode:
    def __init__(self, name):
        self.name = name
        self.type_label = "bone"
        self.category = "noun"
        self.integrity = 0.5       # abstract state
        self.load = 0.0
        self.density = 0.10        # abstract property
        self.strength = 0.10       # new abstract property
        self.max_capacity = 1.0
        self.response_speed = 1.0  # Level 10 by default

    def receive_signal(self, intensity):
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.integrity += delta
        self.integrity = min(1.0, max(0.0, self.integrity))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        # abstract decay
        self.load *= 0.9 ** (1.0 + self.response_speed)
# Define the abstract Chinese royal families (user-facing labels)
chinese_royal_families = [
    "HouseOfZhao", "HouseOfTang", "HouseOfMing", "HouseOfQing"
]

# Extend your existing agent sandbox (assuming you have an agent list)
if not hasattr(sandbox, "agents"):
    sandbox.agents = []

for family in chinese_royal_families:
    class CognitiveAgent:
        def __init__(self, name):
            self.name = name
            self.state = 0.5            # abstract numeric state
            self.attention = 1.0
            self.processing_load = 0.0
            self.max_capacity = 1.0
            self.persuasion_bias = 0.0
            self.response_speed = 1.0   # Level 10 for maximal change

        def receive_signal(self, signal_intensity):
            effective = min(signal_intensity, self.max_capacity - self.processing_load)
            delta = effective * (1.0 + self.persuasion_bias) * (1.0 + self.response_speed)
            self.state += delta
            self.state = min(1.0, max(0.0, self.state))
            self.processing_load += effective * (1.0 + self.response_speed)
            self.attention = max(0.0, 1.0 - self.processing_load)
            leftover = signal_intensity - effective
            return leftover

        def step(self, signal_input=1.0, persuasion=0.0):
            self.persuasion_bias = persuasion
            leftover = self.receive_signal(signal_input)
            self.processing_load *= 0.9 ** (1.0 + self.response_speed)
            return leftover

    sandbox.agents.append(CognitiveAgent(family))
import matplotlib.pyplot as plt

# ------------------ Updated BoneNode with calcium ------------------
class BoneNode:
    def __init__(self, name):
        self.name = name
        self.type_label = "bone"
        self.category = "noun"
        self.integrity = 0.05
        self.load = 0.0
        self.density = 0.05
        self.strength = 0.05
        self.calcium = 0.05
        self.max_capacity = 1.0
        self.response_speed = 1.0  # Level 10

    def receive_signal(self, intensity):
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.integrity += delta
        self.integrity = min(1.0, max(0.0, self.integrity))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        self.load *= 0.9 ** (1.0 + self.response_speed)

# ------------------ Skeletal Sandbox ------------------
class SkeletalSandbox:
    def __init__(self, bone_names):
        self.bones = [BoneNode(name) for name in bone_names]
        self.time = 0
        self.history = []

    def step(self, signal_intensity=1.0):
        self.time += 1
        step_data = {"time": self.time}
        for bone in self.bones:
            leftover = bone.receive_signal(signal_intensity)
            bone.step()
            step_data[f"{bone.name}_integrity"] = bone.integrity
            step_data[f"{bone.name}_density"] = bone.density
            step_data[f"{bone.name}_strength"] = bone.strength
            step_data[f"{bone.name}_calcium"] = bone.calcium
            step_data[f"{bone.name}_load"] = bone.load
            step_data[f"{bone.name}_leftover"] = leftover
        self.history.append(step_data)

# ------------------ Bone Names ------------------
bone_names = [
    "Frontal", "Parietal_L", "Parietal_R", "Occipital", "Temporal_L", "Temporal_R",
    "Sphenoid", "Ethmoid", "Maxilla_L", "Maxilla_R", "Mandible",
    "C1", "C2", "C3", "C4", "C5", "C6", "C7",
    "T1", "T2", "T3", "T4", "T5", "T6", "T7", "T8", "T9", "T10", "T11", "T12",
    "L1", "L2", "L3", "L4", "L5", "Sacrum", "Coccyx",
    "Sternum", "Rib1_L", "Rib1_R", "Rib2_L", "Rib2_R", "Rib3_L", "Rib3_R",
    "Rib4_L", "Rib4_R", "Rib5_L", "Rib5_R", "Rib6_L", "Rib6_R", "Rib7_L", "Rib7_R",
    "Rib8_L", "Rib8_R", "Rib9_L", "Rib9_R", "Rib10_L", "Rib10_R", "Rib11_L", "Rib11_R",
    "Rib12_L", "Rib12_R",
    "Clavicle_L", "Clavicle_R", "Scapula_L", "Scapula_R", "Humerus_L", "Humerus_R",
    "Radius_L", "Radius_R", "Ulna_L", "Ulna_R", "Carpals_L", "Carpals_R",
    "Metacarpals_L", "Metacarpals_R", "Phalanges_L", "Phalanges_R",
    "Pelvis_L", "Pelvis_R", "Femur_L", "Femur_R", "Patella_L", "Patella_R",
    "Tibia_L", "Tibia_R", "Fibula_L", "Fibula_R", "Tarsals_L", "Tarsals_R",
    "Metatarsals_L", "Metatarsals_R", "PhalangesFoot_L", "PhalangesFoot_R"
]

# ------------------ Initialize Sandbox ------------------
skeletal_sandbox = SkeletalSandbox(bone_names)

# ------------------ Run Sandbox ------------------
for _ in range(15):
    skeletal_sandbox.step(signal_intensity=1.0)  # Level 10 input

# ------------------ Visualize Integrity, Density, Strength, Calcium ------------------
plt.figure(figsize=(16, 10))

properties = ["integrity", "density", "strength", "calcium"]
colors = ["blue", "green", "red", "orange"]

for i, prop in enumerate(properties):
    plt.subplot(2, 2, i+1)
    for bone in skeletal_sandbox.bones:
        plt.plot(
            [h["time"] for h in skeletal_sandbox.history],
            [h[f"{bone.name}_{prop}"] for h in skeletal_sandbox.history],
            label=bone.name
        )
    plt.title(f"Bone {prop.capitalize()} over Time")
    plt.xlabel("Time Step")
    plt.ylabel(prop.capitalize())
    plt.grid(True)

plt.tight_layout()
plt.show()
# ------------------ Muscle Node ------------------
class MuscleNode:
    def __init__(self, name):
        self.name = name
        self.tension = 0.05
        self.load = 0.0
        self.max_capacity = 1.0
        self.response_speed = 1.0  # Level 10

    def receive_signal(self, intensity):
        effective = min(intensity, self.max_capacity - self.load)
        delta = effective * (1.0 + self.response_speed)
        self.tension += delta
        self.tension = min(1.0, max(0.0, self.tension))
        self.load += effective * (1.0 + self.response_speed)
        return intensity - effective

    def step(self):
        self.load *= 0.9 ** (1.0 + self.response_speed)

# ------------------ Cognitive Agent ------------------
class CognitiveAgent:
    def __init__(self, name):
        self.name = name
        self.state = 0.5
        self.load = 0.0
        self.response_speed = 1.0

    def send_signal(self, intensity):
        delta = intensity * (1.0 + self.response_speed)
        self.state += delta
        self.state = min(1.0, max(0.0, self.state))
        self.load += intensity
        return intensity * 0.1  # leftover signal to propagate

# ------------------ World-Built Body Sandbox ------------------
class BodySandbox:
    def __init__(self, bone_names, muscle_names, agent_names):
        self.bones = [BoneNode(name) for name in bone_names]
        self.muscles = [MuscleNode(name) for name in muscle_names]
        self.agents = [CognitiveAgent(name) for name in agent_names]
        self.time = 0
        self.history = []

    def step(self):
        self.time += 1
        step_data = {"time": self.time}

        # Agents send signals
        for agent in self.agents:
            signal = agent.send_signal(1.0)

            # Muscles receive agent signals
            for muscle in self.muscles:
                leftover = muscle.receive_signal(signal)
                signal = leftover  # propagate leftover to next

        # Muscles propagate to bones
        for bone in self.bones:
            total_signal = sum([muscle.tension for muscle in self.muscles])
            bone.receive_signal(total_signal)

        # Step muscles
        for muscle in self.muscles:
            muscle.step()

        # Save snapshot
        for bone in self.bones:
            step_data[f"{bone.name}_integrity"] = bone.integrity
            step_data[f"{bone.name}_density"] = bone.density
            step_data[f"{bone.name}_strength"] = bone.strength
            step_data[f"{bone.name}_calcium"] = bone.calcium
        for muscle in self.muscles:
            step_data[f"{muscle.name}_tension"] = muscle.tension

        self.history.append(step_data)

# ------------------ Names ------------------
muscle_names = ["Biceps_L", "Biceps_R", "Quadriceps_L", "Quadriceps_R", "Deltoid_L", "Deltoid_R"]
agent_names = ["Kennedy", "Salazar", "Sussex", "Miller", "HouseOfZhao", "HouseOfTang", "HouseOfMing", "HouseOfQing"]

# ------------------ Initialize Sandbox ------------------
body_sandbox = BodySandbox(bone_names, muscle_names, agent_names)

# ------------------ Run Sandbox ------------------
for _ in range(15):
    body_sandbox.step()

# ------------------ Visualization Example ------------------
plt.figure(figsize=(16, 8))
for bone in body_sandbox.bones[:10]:  # show first 10 bones for clarity
    plt.plot([h["time"] for h in body_sandbox.history],
             [h[f"{bone.name}_integrity"] for h in body_sandbox.history],
             label=f"{bone.name} Integrity")
plt.title("Integrated Skeletal-Muscle Sandbox: Bone Integrity")
plt.xlabel("Time Step")
plt.ylabel("Integrity (0-1)")
plt.grid(True)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
plt.show()
import matplotlib.pyplot as plt

# ------------------ Visualize Full Integrated Body Sandbox ------------------
plt.figure(figsize=(20, 12))

# Properties to visualize
bone_properties = ["integrity", "density", "strength", "calcium"]
muscle_properties = ["tension"]
colors = ["blue", "green", "red", "orange", "purple", "cyan", "magenta", "brown"]

# Plot bones
for i, prop in enumerate(bone_properties):
    plt.subplot(3, 2, i+1)
    for bone in body_sandbox.bones:
        plt.plot(
            [h["time"] for h in body_sandbox.history],
            [h[f"{bone.name}_{prop}"] for h in body_sandbox.history],
            label=bone.name
        )
    plt.title(f"Bones: {prop.capitalize()} over Time")
    plt.xlabel("Time Step")
    plt.ylabel(prop.capitalize())
    plt.grid(True)

# Plot muscles
plt.subplot(3, 2, 5)
for muscle in body_sandbox.muscles:
    plt.plot(
        [h["time"] for h in body_sandbox.history],
        [h[f"{muscle.name}_tension"] for h in body_sandbox.history],
        label=muscle.name
    )
plt.title("Muscle Tension over Time")
plt.xlabel("Time Step")
plt.ylabel("Tension")
plt.grid(True)

# Plot agent states
plt.subplot(3, 2, 6)
for agent in body_sandbox.agents:
    plt.plot(
        [h["time"] for h in body_sandbox.history],
        [agent.state for _ in body_sandbox.history],  # agents update internally
        label=agent.name
    )
plt.title("Agent States over Time")
plt.xlabel("Time Step")
plt.ylabel("State")
plt.grid(True)

plt.tight_layout()
plt.show()
import matplotlib.pyplot as plt

# ------------------ Apply 6x speed to all bones ------------------
speed_multiplier = 6.0
for bone in body_sandbox.bones:
    bone.response_speed *= speed_multiplier  # now 6.0 instead of 1.0

# ------------------ Run Sandbox ------------------
for _ in range(15):
    body_sandbox.step()

# ------------------ Visualization ------------------
plt.figure(figsize=(20, 12))

bone_properties = ["integrity", "density", "strength", "calcium"]
muscle_properties = ["tension"]

# Plot bone properties
for i, prop in enumerate(bone_properties):
    plt.subplot(3, 2, i+1)
    for bone in body_sandbox.bones:
        plt.plot(
            [h["time"] for h in body_sandbox.history],
            [h[f"{bone.name}_{prop}"] for h in body_sandbox.history],
            label=bone.name
        )
    plt.title(f"Bones: {prop.capitalize()} over Time (6× Speed)")
    plt.xlabel("Time Step")
    plt.ylabel(prop.capitalize())
    plt.grid(True)

# Plot muscles
plt.subplot(3, 2, 5)
for muscle in body_sandbox.muscles:
    plt.plot(
        [h["time"] for h in body_sandbox.history],
        [h[f"{muscle.name}_tension"] for h in body_sandbox.history],
        label=muscle.name
    )
plt.title("Muscle Tension over Time")
plt.xlabel("Time Step")
plt.ylabel("Tension")
plt.grid(True)

# Plot agents
plt.subplot(3, 2, 6)
for agent in body_sandbox.agents:
    plt.plot(
        [h["time"] for h in body_sandbox.history],
        [agent.state for _ in body_sandbox.history],  # agent state over time
        label=agent.name
    )
plt.title("Agent States over Time")
plt.xlabel("Time Step")
plt.ylabel("State")
plt.grid(True)

plt.tight_layout()
plt.show()
class BodySandboxDirect:
    def __init__(self, bone_names, muscle_names, agent_names):
        self.bones = [BoneNode(name) for name in bone_names]
        self.muscles = [MuscleNode(name) for name in muscle_names]
        self.agents = [CognitiveAgent(name) for name in agent_names]
        self.time = 0
        self.history = []

    def step(self):
        self.time += 1
        step_data = {"time": self.time}

        # Agents send signals directly to bones
        for agent in self.agents:
            signal = agent.send_signal(1.0)
            for bone in self.bones:
                leftover = bone.receive_signal(signal)
                signal = leftover  # propagate leftover if desired

        # Step muscles (optional, no influence on bones now)
        for muscle in self.muscles:
            muscle.step()

        # Save snapshot for visualization
        for bone in self.bones:
            step_data[f"{bone.name}_integrity"] = bone.integrity
            step_data[f"{bone.name}_density"] = bone.density
            step_data[f"{bone.name}_strength"] = bone.strength
            step_data[f"{bone.name}_calcium"] = bone.calcium
        for muscle in self.muscles:
            step_data[f"{muscle.name}_tension"] = muscle.tension

        self.history.append(step_data)
# Step function remains the same: agents → bones directly
def step_direct_agents_to_bones(sandbox):
    sandbox.time += 1
    step_data = {"time": sandbox.time}

    # Agents send signals directly to bones
    for agent in sandbox.agents:
        signal = agent.send_signal(1.0)
        for bone in sandbox.bones:
            leftover = bone.receive_signal(signal)

    # Step muscles (optional, no influence on bones)
    for muscle in sandbox.muscles:
        muscle.step()

    # Save snapshot for visualization
    for bone in sandbox.bones:
        step_data[f"{bone.name}_integrity"] = bone.integrity
        step_data[f"{bone.name}_density"] = bone.density
        step_data[f"{bone.name}_strength"] = bone.strength
        step_data[f"{bone.name}_calcium"] = bone.calcium
    for muscle in sandbox.muscles:
        step_data[f"{muscle.name}_tension"] = muscle.tension

    sandbox.history.append(step_data)
import matplotlib.pyplot as plt

# ------------------ Direct Agent → Bone Sandbox ------------------
class BodySandboxDirect:
    def __init__(self, bone_names, agent_names):
        self.bones = [BoneNode(name) for name in bone_names]
        self.agents = [CognitiveAgent(name) for name in agent_names]
        self.time = 0
        self.history = []

        # Set all bones to 10x speed
        for bone in self.bones:
            bone.response_speed = 10.0

    def step(self):
        self.time += 1
        step_data = {"time": self.time}

        # Agents send signals directly to bones
        for agent in self.agents:
            signal = agent.send_signal(1.0)
            for bone in self.bones:
                leftover = bone.receive_signal(signal)

        # Save snapshot for visualization
        for bone in self.bones:
            step_data[f"{bone.name}_integrity"] = bone.integrity
            step_data[f"{bone.name}_density"] = bone.density
            step_data[f"{bone.name}_strength"] = bone.strength
            step_data[f"{bone.name}_calcium"] = bone.calcium
        for agent in self.agents:
            step_data[f"{agent.name}_state"] = agent.state

        self.history.append(step_data)

# ------------------ Initialize Sandbox ------------------
direct_sandbox = BodySandboxDirect(bone_names, agent_names)

# ------------------ Run Sandbox ------------------
for _ in range(15):
    direct_sandbox.step()

# ------------------ Visualization ------------------
plt.figure(figsize=(20, 12))

bone_properties = ["integrity", "density", "strength", "calcium"]
colors = ["blue", "green", "red", "orange"]

# Plot bones
for i, prop in enumerate(bone_properties):
    plt.subplot(3, 2, i+1)
    for bone in direct_sandbox.bones:
        plt.plot(
            [h["time"] for h in direct_sandbox.history],
            [h[f"{bone.name}_{prop}"] for h in direct_sandbox.history],
            label=bone.name
        )
    plt.title(f"Bones: {prop.capitalize()} over Time (10× Speed)")
    plt.xlabel("Time Step")
    plt.ylabel(prop.capitalize())
    plt.grid(True)

# Plot agent states
plt.subplot(3, 2, 5)
for agent in direct_sandbox.agents:
    plt.plot(
        [h["time"] for h in direct_sandbox.history],
        [h[f"{agent.name}_state"] for h in direct_sandbox.history],
        label=agent.name
    )
plt.title("Agent States over Time")
plt.xlabel("Time Step")
plt.ylabel("State")
plt.grid(True)

plt.tight_layout()
plt.show()
import matplotlib.pyplot as plt
import numpy as np

# ------------------ Linear Slope Visualization ------------------
plt.figure(figsize=(16, 10))

scaling_factor = 5.0  # how sharply slope increases as saturation rises

for bone in direct_sandbox.bones:
    times = np.array([h["time"] for h in direct_sandbox.history])
    saturations = np.array([h[f"{bone.name}_integrity"] for h in direct_sandbox.history])

    # Calculate slope dynamically: slope increases with saturation
    slopes = saturations * scaling_factor
    y_values = [saturations[0]]  # start at initial value

    # Calculate y = y_prev + slope * delta_x
    for i in range(1, len(times)):
        delta_x = times[i] - times[i-1]
        y_new = y_values[-1] + slopes[i] * delta_x
        y_new = min(1.0, y_new)  # cap at saturation 1.0
        y_values.append(y_new)

    plt.plot(times, y_values, label=bone.name)

plt.title("Skeletal System Integrity with Accelerating Slope (y = mx + b)")
plt.xlabel("Time Step")
plt.ylabel("Bone Integrity (0-1)")
plt.grid(True)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')
plt.tight_layout()
plt.show()
import matplotlib.pyplot as plt
import numpy as np

# ------------------ Accelerating Slope for All Bone Properties ------------------
plt.figure(figsize=(20, 15))

bone_properties = ["integrity", "density", "strength", "calcium"]
scaling_factor = 5.0  # controls how sharply slope increases as saturation rises

for i, prop in enumerate(bone_properties):
    plt.subplot(2, 2, i+1)

    for bone in direct_sandbox.bones:
        times = np.array([h["time"] for h in direct_sandbox.history])
        saturations = np.array([h[f"{bone.name}_{prop}"] for h in direct_sandbox.history])

        # Calculate dynamic slope: slope increases with saturation
        slopes = saturations * scaling_factor
        y_values = [saturations[0]]  # start at initial value

        # Generate y = y_prev + slope * delta_x
        for j in range(1, len(times)):
            delta_x = times[j] - times[j-1]
            y_new = y_values[-1] + slopes[j] * delta_x
            y_new = min(1.0, y_new)  # cap at 1.0
            y_values.append(y_new)

        plt.plot(times, y_values, label=bone.name)

    plt.title(f"Bones: {prop.capitalize()} with Accelerating Slope")
    plt.xlabel("Time Step")
    plt.ylabel(prop.capitalize())
    plt.grid(True)

plt.tight_layout()
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')
plt.show()
import matplotlib.pyplot as plt
import numpy as np
from matplotlib.animation import FuncAnimation

# ------------------ Animation Setup ------------------
bone_properties = ["integrity", "density", "strength", "calcium"]
scaling_factor = 5.0  # controls slope acceleration

fig, axes = plt.subplots(2, 2, figsize=(18, 12))
axes = axes.flatten()

lines = {prop: {} for prop in bone_properties}

for i, prop in enumerate(bone_properties):
    ax = axes[i]
    ax.set_xlim(0, len(direct_sandbox.history))
    ax.set_ylim(0, 1.05)
    ax.set_title(f"Bones: {prop.capitalize()} with Accelerating Slope")
    ax.set_xlabel("Time Step")
    ax.set_ylabel(prop.capitalize())
    ax.grid(True)

    for bone in direct_sandbox.bones:
        line, = ax.plot([], [], label=bone.name)
        lines[prop][bone.name] = line
    ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')

# ------------------ Animation Function ------------------
def animate(frame):
    for i, prop in enumerate(bone_properties):
        for bone in direct_sandbox.bones:
            times = np.array([h["time"] for h in direct_sandbox.history[:frame+1]])
            saturations = np.array([h[f"{bone.name}_{prop}"] for h in direct_sandbox.history[:frame+1]])
            slopes = saturations * scaling_factor
            y_values = [saturations[0]]
            for j in range(1, len(times)):
                delta_x = times[j] - times[j-1]
                y_new = y_values[-1] + slopes[j] * delta_x
                y_new = min(1.0, y_new)
                y_values.append(y_new)
            lines[prop][bone.name].set_data(times, y_values)
    return [line for prop_lines in lines.values() for line in prop_lines.values()]

# ------------------ Run Animation ------------------
anim = FuncAnimation(fig, animate, frames=len(direct_sandbox.history), interval=500, blit=True)
plt.tight_layout()
plt.show()
# ------------------ Animation Function with b = 0.1 ------------------
def animate(frame):
    for i, prop in enumerate(bone_properties):
        for bone in direct_sandbox.bones:
            times = np.array([h["time"] for h in direct_sandbox.history[:frame+1]])
            saturations = np.array([h[f"{bone.name}_{prop}"] for h in direct_sandbox.history[:frame+1]])
            
            # Set initial value b = 0.1
            y_values = [0.1]  

            # Dynamic slope
            slopes = saturations * scaling_factor
            for j in range(1, len(times)):
                delta_x = times[j] - times[j-1]
                y_new = y_values[-1] + slopes[j] * delta_x
                y_new = min(1.0, y_new)  # cap at saturation 1.0
                y_values.append(y_new)

            lines[prop][bone.name].set_data(times, y_values)
    return [line for prop_lines in lines.values() for line in prop_lines.values()]
import matplotlib.pyplot as plt
import numpy as np
from matplotlib.animation import FuncAnimation

# ------------------ Animation Setup ------------------
bone_properties = ["integrity", "density", "strength", "calcium"]
scaling_factor = 5.0  # slope acceleration factor

fig, axes = plt.subplots(2, 2, figsize=(18, 12))
axes = axes.flatten()

lines = {prop: {} for prop in bone_properties}

for i, prop in enumerate(bone_properties):
    ax = axes[i]
    ax.set_xlim(0, len(direct_sandbox.history))
    ax.set_ylim(0, 1.05)
    ax.set_title(f"Bones: {prop.capitalize()} with Accelerating Slope")
    ax.set_xlabel("Time Step")
    ax.set_ylabel(prop.capitalize())
    ax.grid(True)

    for bone in direct_sandbox.bones:
        line, = ax.plot([], [], label=bone.name)
        lines[prop][bone.name] = line
    ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left', fontsize='small')

# ------------------ Animation Function with b = 0.8 ------------------
def animate(frame):
    for i, prop in enumerate(bone_properties):
        for bone in direct_sandbox.bones:
            times = np.array([h["time"] for h in direct_sandbox.history[:frame+1]])
            saturations = np.array([h[f"{bone.name}_{prop}"] for h in direct_sandbox.history[:frame+1]])

            # Initial value b = 0.8
            y_values = [0.8]

            # Dynamic slope: slope increases as property approaches 1.0
            slopes = saturations * scaling_factor
            for j in range(1, len(times)):
                delta_x = times[j] - times[j-1]
                y_new = y_values[-1] + slopes[j] * delta_x
                y_new = min(1.0, y_new)  # cap at 1.0 saturation
                y_values.append(y_new)

            lines[prop][bone.name].set_data(times, y_values)
    return [line for prop_lines in lines.values() for line in prop_lines.values()]

# ------------------ Run Animation ------------------
anim = FuncAnimation(fig, animate, frames=len(direct_sandbox.history), interval=500, blit=True)
plt.tight_layout()
plt.show()
