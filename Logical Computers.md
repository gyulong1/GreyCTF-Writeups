We first examine the source code provided,

```
import torch

def tensorize(s : str) -> torch.Tensor:
  return torch.Tensor([(1 if (ch >> i) & 1 == 1 else -1) for ch in list(map(ord, s)) for i in range(8)])

class NeuralNetwork(torch.nn.Module):
  def __init__(self, in_dimension, mid_dimension, out_dimension=1):
    super(NeuralNetwork, self).__init__()
    self.layer1 = torch.nn.Linear(in_dimension, mid_dimension)
    self.layer2 = torch.nn.Linear(mid_dimension, out_dimension)

  def step_activation(self, x : torch.Tensor) -> torch.Tensor:
    x[x <= 0] = -1
    x[x >  0] = 1
    return x

  def forward(self, x : torch.Tensor) -> int:
    x = self.layer1(x)
    x = self.step_activation(x)
    x = self.layer2(x)
    x = self.step_activation(x)
    return int(x)

flag = input("Enter flag: ")
in_data = tensorize(flag)
in_dim	= len(in_data)

model = NeuralNetwork(in_dim, 1280)
model.load_state_dict(torch.load("model.pth"))

if model(in_data) == 1:
	print("Yay correct! That's the flag!")
else:
	print("Awww no...")
```
A model file `model.pth` is also provided which is a pre-trained model that recognises the flag. The problem is to input the flag which will be verified by the model. However, when we tested with random strings as the input, `model(in_data)` always returns `-1` which isn't helpful as it doesn't indicate if we are getting closer or further away from the flag. We are also able to obtain the flag length as only strings with 20 characters can be used.

The problem lies in the `step_activation` function which is currently a binary step function, meaning that we cannot obtain useful feedback as the output will always be `-1` is the flag is wrong. Read more about activation functions [here](https://towardsdatascience.com/getting-to-know-activation-functions-in-neural-networks-125405b67428).

Hence, we change the function to return `x` itself instead of a binary value

```
def step_activation(self, x : torch.Tensor) -> torch.Tensor:
    return x
```

We can test with different flags to see that the model value does indeed vary. The string that is closest to the flag should thus have a higher model and we can bruteforce the flag character by character.

```
model = NeuralNetwork(in_dim, 1280)
model.load_state_dict(torch.load("model.pth"))

test = "grey{aaaaaaaaaaaaaa}"

def split(word):
    return [char for char in word]

// stores flag
ans = []

// for each character within the brackets
for i in range(5, 19):
    data = []
    // iterate through each possible ASCII character
    for c in range(33, 125):
        acc = split(test)
        acc[i] = chr(c)
        data.append(model(tensorize("".join(acc))))
    // find and store the character with the largest value
    index = data.index(max(data))
    ans.append(chr(index + 33))

print("grey{" + "".join(ans) + "}")
```

This gives us the flag `grey{sM0rT_mAch1nE5}`.