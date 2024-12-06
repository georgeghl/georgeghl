# git 设置和取消代理

```powershell
# 设置
git config --global https.proxy 127.0.0.1:7890
git config --global http.proxy 127.0.0.1:7890

# 取消
git config --global --unset http.proxy
git config --global --unset https.proxy
```

