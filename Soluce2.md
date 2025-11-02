# 2e solution

# sur Mint désactiver le powerdown graphic


```bash
gsettings set org.cinnamon.desktop.lockdown disable-log-out true
```

# redéfinir poweroff et reboot

```bash
alias poweroff='read -p "⚠️ Voulez-vous vraiment éteindre ? pensez à suspendre les VM [o/N] " reponse; [[ $reponse == "o" || $reponse == "O" ]] && sudo /usr/lib/klibc/bin/poweroff'
alias reboot='read -p "⚠️ Voulez-vous vraiment redémarrer ? pensez à suspendre les VM [o/N] " reponse; [[ $reponse == "o" || $reponse == "O" ]] && sudo /usr/lib/klibc/bin/reboot'
alias vagrant_down='vagrant suspend'
alias vagrant_resume='vagrant resume'
alias vagrant_suspend='vagrant suspend'
alias vagrant_up='vagrant resume'
```