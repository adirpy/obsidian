
## Catalina下提示文件损坏

```bash
sudo xattr -r -d com.apple.quarantine /Application/xxx.app
```

## Mac Mini进入Recovery Mode

1. Launch the Terminal.
2. Type in:```sudo nvram "recovery-boot-mode=unused"```
3. Type in:```sudo shutdown -r now```
    (this will reboot your Mac into the Recovery drive)
4. You’ll boot to your Recovery drive
5. When you’re finished, open the Terminal from the menu.
6. Type in:```nvram boot-args=""```
7. Restart your Mac and you’ll boot back to your main drive
