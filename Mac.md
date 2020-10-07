# MAC
## Catalina Upgrade

- Interactive shell is now zsh
- https://support.apple.com/en-us/HT208050
- https://tidbits.com/2019/12/08/resources-for-adapting-to-zsh-in-catalina/



## Cask

- Out-dated to an extent - https://sourabhbajaj.com/mac-setup/Homebrew/Cask.html

##  What program is using port 8080

```bash
lsof -i :8080 | grep LISTEN
# output
java    42975 mkyong   

ps -ef 42975
kill 42975
```

