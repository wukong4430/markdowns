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

