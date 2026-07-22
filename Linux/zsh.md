**未来维护提醒**：每次 `mise self-update` 大版本升级后，如果 Tab 补全突然失效，只需重新执行：

```
mise completion zsh > ~/.local/share/mise/completions/_mise
rm -f ~/.oh-my-zsh/cache/.zcompdump*
```

