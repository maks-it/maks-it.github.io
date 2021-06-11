Add option -m PEM into your ssh-keygen command. For example, you can run 

```bash
ssh-keygen -m PEM -t rsa -b 4096 -C "your_email@example.com"
```

to force ssh-keygen to export as PEM format.