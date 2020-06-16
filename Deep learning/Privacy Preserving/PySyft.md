# 1、Introduction



## 1.1 About pysyft

**A privacy preserving, decentralized deep learning tool.**  We run all the model with the data that does not exist on the local machine.

## 1.2 How to use data remotely (with virtualWorker)



### Send tensors to remote workers

```python
bob = sy.VirtualWorker(hook, id="bob")
x = torch.tensor([1,2,3,4,5])
y = torch.tensor([1,1,1,1,1])
x_ptr = x.send(bob)
y_ptr = y.send(bob)
bob._objects
```

- Here we create two pointers to tensor, pointers to tensor (p2t) does not actually hold data themselves. Instead, they simply contain metadata about a tensor (with data) stored on another machine. 



- The tensors 【x,y】 are on the remote machine (We manually send it to bob). Meanwhile, the p2t is owned by local worker (me). 



### Get tensors from remote workers

Just like we can call .send() on a tensor, we can call .get() on a pointer to a tensor to get it back!!!

```python
x_ptr.get()
```

Once we get tensors from remote machine, the remote one does not have it immediately.

```python
bob.clear_objects()
bob = sy.VirtualWorker(hook, id="bob")

x = torch.tensor([1,2,3,4,5])
y = torch.tensor([1,1,1,1,1])
x_ptr = x.send(bob)
y_ptr = y.send(bob)

# z = x_ptr + x_ptr


print(bob._objects)
print(x_ptr.location==bob)
print(x_ptr.owner)
sy.local_worker==x_ptr.owner

x_ptr.get()
bob._objects

>>>>>>>
{27081345368: tensor([1, 2, 3, 4, 5]), 66204479032: tensor([1, 1, 1, 1, 1])}
True
<VirtualWorker id:me #objects:0>
{66204479032: tensor([1, 1, 1, 1, 1])}
```



### Compute remotely, get the result.

```python
x_ptr = torch.tensor([1,2,3,4,5]).send(bob)
y_ptr = torch.tensor([5,4,3,2,1]).send(bob)

z_ptr = x_ptr + y_ptr ;

z_ptr.get()

>>>>>
tensor([6, 6, 6, 6, 6])
```





### Toy model

- Train on local machine

```python
# A Toy Dataset
data = torch.tensor([[0,0],[0,1],[1,0],[1,1.]])
target = torch.tensor([[0],[0],[1],[1.]])

# A Toy Model
model = nn.Linear(2,1)

def train():
    # Training Logic
    opt = optim.SGD(params=model.parameters(),lr=0.1)
    for iter in range(20):

        # 1) erase previous gradients (if they exist)
        opt.zero_grad()

        # 2) make a prediction
        pred = model(data)

        # 3) calculate how much we missed
        loss = ((pred - target)**2).sum()

        # 4) figure out which weights caused us to miss
        loss.backward()

        # 5) change those weights
        opt.step()

        # 6) print our progress
        print(loss.data)

train()
```

```bah
tensor(1.3321)
tensor(0.3157)
tensor(0.1670)
tensor(0.1083)
tensor(0.0726)
tensor(0.0491)
tensor(0.0335)
tensor(0.0230)
tensor(0.0159)
tensor(0.0111)
tensor(0.0078)
tensor(0.0055)
tensor(0.0039)
tensor(0.0028)
tensor(0.0021)
tensor(0.0015)
tensor(0.0011)
tensor(0.0008)
tensor(0.0006)
tensor(0.0004)
```



- Train on remote machine

```python
hook = sy.TorchHook(torch)

# create a couple workers
bob.clear_objects()

bob = sy.VirtualWorker(hook, id="bob")
alice = sy.VirtualWorker(hook, id="alice")

# A Toy Dataset
data = torch.tensor([[0,0],[0,1],[1,0],[1,1.]], requires_grad=True, dtype=torch.float)
target = torch.tensor([[0],[0],[1],[1.]], requires_grad=True, dtype=torch.float)

# get pointers to training data on each worker by
# sending some training data to bob and alice
data_bob = data[0:2]
target_bob = target[0:2]

data_alice = data[2:]
target_alice = target[2:]

# Iniitalize A Toy Model
model = nn.Linear(2,1)

data_bob = data_bob.send(bob)
data_alice = data_alice.send(alice)
target_bob = target_bob.send(bob)
target_alice = target_alice.send(alice)

# organize pointers into a list
datasets = [(data_bob,target_bob),(data_alice,target_alice)]
```

1. create several workers
2. prepare dataset
3. send dataset to workers
4. assign dataset to pointers



```python
from syft.federated.floptimizer import Optims
workers = ['bob', 'alice']
optims = Optims(workers, optim=optim.Adam(params=model.parameters(),lr=0.1))

def train_federated():
    # Training Logic
    for iter in range(10):
        
        # NEW) iterate through each worker's dataset
        for data,target in datasets:
            
            # NEW) send model to correct worker
            model.send(data.location)
            
            #Call the optimizer for the worker using get_optim
            opt = optims.get_optim(data.location.id)
            #print(data.location.id)

            # 1) erase previous gradients (if they exist)
            opt.zero_grad()

            # 2) make a prediction
            pred = model(data)

            # 3) calculate how much we missed
            loss = ((pred - target)**2).sum()

            # 4) figure out which weights caused us to miss
            loss.backward()

            # 5) change those weights
            opt.step()
            
            # NEW) get model (with gradients)
            model.get()

            # 6) print our progress
            print(loss.get()) # NEW) slight edit... need to call .get() on loss\
    
# federated averaging
```

5. import optimizer from syft
6. **define optim with workers and parameters.**
7. **Create training process**
    - **get data from datasets (pointers)**
    - **send model to the right worker ('cause data is on the worker)**
    - **Call the optimizer for the worker using get_optim**
    - normal train
    - **get model and get loss**





### Shortcomings in this example

When we get model from remote workers, it is easily to obtain the gradients and then restore the original data. (Not really safe.)





## 1.3 Average gradients

To prevent restoring data from gradients, we want to average the gradients **before** calling .get(). That way, we won't ever see anyone's exact gradient (thus better protecting their privacy!!!)



### Send pointer to a remote machine

```python
bob.clear_objects()
alice.clear_objects()

x = torch.tensor([1,2,3,4])


x_ptr = x.send(bob)
x_ptr

print(x_ptr.owner)

pointer_to_x_ptr = x_ptr.send(alice)

print(pointer_to_x_ptr.owner)
```

```
<VirtualWorker id:me #objects:0>
<VirtualWorker id:me #objects:0>
```

**The owner is me.**

```python
print(bob._objects)

print(alice._objects)
```

```
{10120575439: tensor([1, 2, 3, 4])}
{6896548230: (Wrapper)>[PointerTensor | alice:6896548230 -> bob:10120575439]}
```

bob owns a tensor while Alice owns a pointer (points at bob)



### Send data directly to a remote machine

```python
x = torch.tensor([1,2,3,4,5]).send(bob)
print('  bob:', bob._objects)
print('alice:',alice._objects)
print(x)
x = x.move(alice)
print('  bob:', bob._objects)
print('alice:',alice._objects)
x
```

```bash
bob: {56883919075: tensor([1, 2, 3, 4, 5])}
alice: {}
(Wrapper)>[PointerTensor | me:89797370294 -> bob:56883919075]
  bob: {}
alice: {56883919075: tensor([1, 2, 3, 4, 5])}
(Wrapper)>[PointerTensor | me:89797370294 -> alice:56883919075]
```





### Train two models and average weights

```python
iterations = 10
worker_iters = 5

for a_iter in range(iterations):
    
    bobs_model = model.copy().send(bob)
    alices_model = model.copy().send(alice)

    bobs_opt = optim.SGD(params=bobs_model.parameters(),lr=0.1)
    alices_opt = optim.SGD(params=alices_model.parameters(),lr=0.1)

    for wi in range(worker_iters):

        # Train Bob's Model
        bobs_opt.zero_grad()
        bobs_pred = bobs_model(bobs_data)
        bobs_loss = ((bobs_pred - bobs_target)**2).sum()
        bobs_loss.backward()

        bobs_opt.step()
        bobs_loss = bobs_loss.get().data

        # Train Alice's Model
        alices_opt.zero_grad()
        alices_pred = alices_model(alices_data)
        alices_loss = ((alices_pred - alices_target)**2).sum()
        alices_loss.backward()

        alices_opt.step()
        alices_loss = alices_loss.get().data
    
    alices_model.move(secure_worker)
    bobs_model.move(secure_worker)
    with torch.no_grad():
        model.weight.set_(((alices_model.weight.data + bobs_model.weight.data) / 2).get())
        model.bias.set_(((alices_model.bias.data + bobs_model.bias.data) / 2).get())
    
    print("Bob:" + str(bobs_loss) + " Alice:" + str(alices_loss))
```

1. send copy of model to workers
2. create optims
3. train as usual
    - Here bobs_model is pointer of model; bobs_data is pointer of data
    - loss.get()
4. **Send two models (trained) to a secure worker**
5. average weights
6. **Update the model locally**



***Finally, we are able to use the updated model on the data locally.***





## 1.4 Demo: MNIST

List all differences from the single machine.

### Data loading and sending to workers

```python
federated_train_loader = sy.FederatedDataLoader( # <-- this is now a FederatedDataLoader 
    datasets.MNIST('../data', train=True, download=True,
                   transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize((0.1307,), (0.3081,))
                   ]))
    .federate((bob, alice)), # <-- NEW: we distribute the dataset across all the workers, it's now a FederatedDataset
    batch_size=args.batch_size, shuffle=True, **kwargs)

test_loader = torch.utils.data.DataLoader(
    datasets.MNIST('../data', train=False, transform=transforms.Compose([
                       transforms.ToTensor(),
                       transforms.Normalize((0.1307,), (0.3081,))
                   ])),
    batch_size=args.test_batch_size, shuffle=True, **kwargs)

```

- DataLoader now is 【sy.FederatedDataLoader】(Only Train data, because we use test data locally)
- **Need to use method 【federate】to distribute the dataset across all the workers.**
- 

### Define the train and test functions

```python
def train(args, model, device, federated_train_loader, optimizer, epoch):
    model.train()
    for batch_idx, (data, target) in enumerate(federated_train_loader): # <-- now it is a distributed dataset
        model.send(data.location) # <-- NEW: send the model to the right location
        data, target = data.to(device), target.to(device)
        optimizer.zero_grad()
        output = model(data)
        loss = F.nll_loss(output, target)
        loss.backward()
        optimizer.step()
        model.get() # <-- NEW: get the model back
        if batch_idx % args.log_interval == 0:
            loss = loss.get() # <-- NEW: get the loss back
            print('Train Epoch: {} [{}/{} ({:.0f}%)]\tLoss: {:.6f}'.format(
                epoch, batch_idx * args.batch_size, len(federated_train_loader) * args.batch_size,
                100. * batch_idx / len(federated_train_loader), loss.item()))
```

- Send model to remote location
- get model updated
- get loss back



The test function does not change!

```python
def test(args, model, device, test_loader):
    model.eval()
    test_loss = 0
    correct = 0
    with torch.no_grad():
        for data, target in test_loader:
            data, target = data.to(device), target.to(device)
            output = model(data)
            test_loss += F.nll_loss(output, target, reduction='sum').item() # sum up batch loss
            pred = output.argmax(1, keepdim=True) # get the index of the max log-probability 
            correct += pred.eq(target.view_as(pred)).sum().item()

    test_loss /= len(test_loader.dataset)

    print('\nTest set: Average loss: {:.4f}, Accuracy: {}/{} ({:.0f}%)\n'.format(
        test_loss, correct, len(test_loader.dataset),
        100. * correct / len(test_loader.dataset)))

```



Training Process

```python
%%time
model = Net().to(device)
optimizer = optim.SGD(model.parameters(), lr=args.lr) # TODO momentum is not supported at the moment

for epoch in range(1, args.epochs + 1):
    train(args, model, device, federated_train_loader, optimizer, epoch)
    test(args, model, device, test_loader)

if (args.save_model):
    torch.save(model.state_dict(), "mnist_cnn.pt")
```





# 2、Plans

A Plan is intended to store a sequence of torch operations, just like a function, but it allows to send this sequence of operations to remote workers and to keep a reference to it. 



## Reason

We use a plan to reduce unnecessary operations.



## Usage 1

- Create a plan:

```python
@sy.func2plan(args_shape=[(-1,),(-1,)])
def calcu(x_ptr, y_ptr):
	z_ptr = x_ptr + y_ptr
	m_ptr = z_ptr / 2
	return m_ptr
```

- send a plan to remote machine

```python
pointer_plan = calcu.send(alice)
```

- Invoke the plan

```python
pointer_result = pointer_plan(x_ptr,y_ptr)
```



## Usage 2 (by extends)

- Create a class (network)

```python
class Net(sy.Plan):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(2, 3)
        self.fc2 = nn.Linear(3, 2)

    def forward(self, x):
        x = F.relu(self.fc1(x))
        x = self.fc2(x)
        return F.log_softmax(x, dim=0)

```

- Build

```python
net = Net()
net.build(torch.tensor([1.,2.]))
```

- Send and execute

```python
ptr_net = net.send(bob)
result_ptr = ptr_net(input_data)
result_ptr.get()
```



