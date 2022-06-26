# Fork Bomb

https://www.imperva.com/learn/ddos/fork-bomb/#:~:text=A%20fork%20bomb%20(also%20known,to%20respond%20to%20any%20input.

## cpu

Scream void until cpu reaches it limit. 

```bash
yes > /dev/null
```

## memory

Fill up the memory until there's no more space.

```bash
yes | tr \\n x | head -c 1048576000 | grep n 
```
