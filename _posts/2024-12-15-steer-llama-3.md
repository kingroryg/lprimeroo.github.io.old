---
layout: post
---

I've been experimenting with Llama-3 lately, and one of the most fascinating aspects I've explored is the concept of steering vectors. These allow us to guide the model's outputs in specific directions without fine-tuning the entire model. In this post, I'll walk through how to create and apply steering vectors to Llama-3 using Python, and share some insights from my experimentation.

## What are Steering Vectors?

Before diving into implementation, let's clarify what steering vectors actually are. In the context of large language models like Llama-3, a steering vector is essentially a direction in the model's activation space that, when added to the model's hidden states, biases the generation toward certain attributes or qualities.

The concept builds on the idea that we can identify directions in the model's latent space that correspond to specific attributes (like politeness, formality, or technical depth). By adding or subtracting along these directions, we can "steer" the model's outputs.

In this post, we'll focus on creating a politeness steering vector, which allows us to guide the model between polite, courteous responses and more direct, concise ones. This has practical applications in tailoring content to different audiences and contexts.

## Setting Up the Environment

First, we need to set up our environment with the necessary libraries:

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
import numpy as np
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

# Set device
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"Using device: {device}")
```

We'll be using HuggingFace's transformers library to work with Llama-3, along with NumPy and scikit-learn for the vector manipulations.

## Loading the Llama-3 Model

Next, let's load the Llama-3 model:

```python
model_name = "meta-llama/Llama-3-8b"  # You can use other variants based on your needs

# Load model and tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,  # Use half precision to save memory
    device_map="auto"
)

print(f"Model loaded: {model_name}")
```

Make sure you have the necessary permissions to access the model. For Llama-3, you'll likely need to request access through Meta's official channels and configure your HuggingFace token appropriately.

## Collecting Activations for Contrasting Examples

The foundation of creating a steering vector is collecting model activations for contrasting examples. For example, if we want to create a "technical vs. casual" steering vector, we need examples of both technical and casual text.

```python
def get_hidden_states(text, layer_idx=-1):
    """Extract hidden states from a specific layer for the given text."""
    inputs = tokenizer(text, return_tensors="pt").to(device)
    
    # We'll use hooks to capture the hidden states
    hidden_states = []
    
    def hook_fn(module, input, output):
        hidden_states.append(output[0].detach())
    
    # Register the hook on the specified layer
    target_layer = model.model.layers[layer_idx]
    hook = target_layer.register_forward_hook(hook_fn)
    
    # Forward pass
    with torch.no_grad():
        outputs = model(**inputs)
    
    # Remove the hook
    hook.remove()
    
    return hidden_states[0], inputs.input_ids
```

Now, let's collect examples for our contrasting attributes:

```python
# Example pairs (polite vs. direct explanations)
polite_examples = [
    "I would greatly appreciate it if you could explain the concept of machine learning to me when you have a moment.",
    "Would you be so kind as to share your thoughts on this approach? Thank you for your consideration.",
    "If it's not too much trouble, could you please clarify this point? I'd be most grateful for your insights."
]

direct_examples = [
    "Explain machine learning to me now.",
    "Tell me what you think about this approach.",
    "Clarify this point immediately."
]

# Collect hidden states
polite_states = []
direct_states = []

for text in polite_examples:
    states, _ = get_hidden_states(text)
    polite_states.append(states.mean(dim=1))  # Average over sequence length

for text in direct_examples:
    states, _ = get_hidden_states(text)
    direct_states.append(states.mean(dim=1))  # Average over sequence length

# Convert to tensors
polite_tensor = torch.cat(polite_states)
direct_tensor = torch.cat(direct_states)

print(f"Polite tensor shape: {polite_tensor.shape}")
print(f"Direct tensor shape: {direct_tensor.shape}")
```

## Computing the Steering Vector

With our activations collected, we can now compute the steering vector:

```python
# Compute centroids (average representations)
polite_centroid = polite_tensor.mean(dim=0)
direct_centroid = direct_tensor.mean(dim=0)

# The steering vector is the difference between centroids
# Here we create a politeness steering vector (from direct to polite)
steering_vector = polite_centroid - direct_centroid

# Normalize the vector
steering_vector = steering_vector / torch.norm(steering_vector)

print(f"Politeness steering vector shape: {steering_vector.shape}")
```

This steering vector now represents the direction from "casual" to "technical" in the model's activation space. To steer toward more technical outputs, we would add this vector to the activations; to steer toward more casual outputs, we would subtract it.

## Visualizing the Steering Vector

It can be helpful to visualize our examples and the steering vector to ensure they make sense:

```python
# Combine all examples
all_states = torch.cat([polite_tensor, direct_tensor])

# Use PCA to reduce to 2 dimensions for visualization
pca = PCA(n_components=2)
reduced_states = pca.fit_transform(all_states.cpu().numpy())

# Split back into polite and direct
polite_reduced = reduced_states[:len(polite_tensor)]
direct_reduced = reduced_states[len(polite_tensor):]

# Plot
plt.figure(figsize=(10, 8))
plt.scatter(polite_reduced[:, 0], polite_reduced[:, 1], c='blue', label='Polite')
plt.scatter(direct_reduced[:, 0], direct_reduced[:, 1], c='red', label='Direct')

# Add centroids
polite_centroid_reduced = pca.transform(polite_centroid.unsqueeze(0).cpu().numpy())[0]
direct_centroid_reduced = pca.transform(direct_centroid.unsqueeze(0).cpu().numpy())[0]

plt.scatter(polite_centroid_reduced[0], polite_centroid_reduced[1], c='darkblue', s=200, marker='*', label='Polite Centroid')
plt.scatter(direct_centroid_reduced[0], direct_centroid_reduced[1], c='darkred', s=200, marker='*', label='Direct Centroid')

# Add an arrow for the steering vector
arrow_start = direct_centroid_reduced
arrow_end = polite_centroid_reduced
plt.arrow(arrow_start[0], arrow_start[1], 
          arrow_end[0] - arrow_start[0], arrow_end[1] - arrow_start[1],
          head_width=0.05, head_length=0.1, fc='red', ec='red', label='Steering Vector')

plt.title('PCA Visualization of Politeness Steering Vector')
plt.xlabel('Principal Component 1')
plt.ylabel('Principal Component 2')
plt.legend()
plt.grid(True)
plt.savefig('politeness_steering_vector_visualization.png')
plt.show()
```

## Applying the Steering Vector During Generation

Now for the interesting part: applying our steering vector during text generation:

```python
def generate_with_steering(prompt, steering_vector, layer_idx=-1, alpha=1.0, max_length=100):
    """Generate text with steering vector applied at the specified layer."""
    inputs = tokenizer(prompt, return_tensors="pt").to(device)
    input_ids = inputs.input_ids
    
    # Function to modify hidden states during generation
    def steering_hook(module, input_states, output_states):
        # Apply steering to the hidden states
        modified_states = output_states[0] + alpha * steering_vector.unsqueeze(0).unsqueeze(0)
        return (modified_states,) + output_states[1:]
    
    # Register the hook
    target_layer = model.model.layers[layer_idx]
    hook = target_layer.register_forward_hook(steering_hook)
    
    # Generate text
    output_ids = model.generate(
        input_ids,
        max_length=max_length,
        num_return_sequences=1,
        temperature=0.7,
        top_p=0.9,
    )
    
    # Remove the hook
    hook.remove()
    
    # Decode the output
    output_text = tokenizer.decode(output_ids[0], skip_special_tokens=True)
    return output_text
```

Let's test it with different steering intensities:

```python
prompt = "I need your feedback on my project proposal. "

# Generate with different steering intensities
print("Without steering:")
print(tokenizer.decode(model.generate(
    tokenizer(prompt, return_tensors="pt").to(device).input_ids,
    max_length=100,
    num_return_sequences=1,
    temperature=0.7,
    top_p=0.9,
)[0], skip_special_tokens=True))
print("\n" + "-"*50 + "\n")

print("With polite steering (alpha=2.0):")
print(generate_with_steering(prompt, steering_vector, alpha=2.0))
print("\n" + "-"*50 + "\n")

print("With direct steering (alpha=-2.0):")
print(generate_with_steering(prompt, steering_vector, alpha=-2.0))
```

## Handling Gotchas and Limitations

As with many ML techniques, there are some gotchas to be aware of:

1. **Layer Selection**: Different layers in the model encode different types of information. Experiment with applying steering vectors to different layers to see what works best for your use case. For politeness steering, I've found that later layers tend to work better as they capture more high-level semantic features.

2. **Alpha Calibration**: The steering intensity (alpha) needs careful tuning. Too high, and your outputs become exaggerated or unnatural; too low, and the effect is negligible. For politeness, an overly high alpha might result in excessively formal or obsequious text.

3. **Example Quality**: The quality of your steering vector depends on how well your examples represent the attributes you're trying to capture. For our politeness vector, we used examples that clearly contrast direct commands with polite requests, avoiding confounding factors like differences in content or length.

4. **Vector Stability**: The stability of your steering vector may vary depending on the size of your example set. More examples generally lead to more stable vectors. I recommend at least 5-10 examples for each end of the spectrum you're trying to capture.

5. **Model Compatibility**: This approach should work for Llama-3, but may need adjustments for other architectures.

## Advanced Applications

Beyond politeness steering, you can create vectors for a wide range of attributes:

- Helpfulness vs. brevity
- Creative vs. analytical
- Domain-specific expertise (e.g., medical, legal, technical)
- Different writing styles or tones
- Emotional attributes (empathetic, enthusiastic, neutral)

You can even combine multiple steering vectors with different weights to achieve more nuanced control over the model's outputs. For example, you might want a response that's both polite and technical, or direct but creative.

## Conclusion

Steering vectors provide a powerful and flexible way to guide Llama-3's outputs without the need for fine-tuning. They give us a more granular level of control and allow for on-the-fly adjustments to the model's behavior.

In my experimentation with politeness vectors, I've found that this approach can be particularly valuable in customer-facing applications, where adapting the tone to different contexts is crucial. A polite, deferential tone works well for formal business communications, while a more direct tone might be appropriate for technical instructions or emergency information.

The politeness vector we created here is just one example of how we can steer language models like Llama-3. The same technique can be applied to a wide range of attributes and characteristics, allowing for fine-grained control over the model's outputs.

While this technique is still evolving, it represents an important step toward more controllable and useful language models. As we continue to explore the latent spaces of these models, I expect we'll discover even more powerful ways to guide their behavior.

If you're working with Llama-3 or other language models, I'd encourage you to experiment with steering vectors and share your findings. The field is still young, and there's plenty of unexplored territory to discover.
