#Ubuntu install Yahei and consolas font

```
sudo mkdir -p /usr/share/fonts/vista/ 
sudo cp ./YaHei.Consolas.1.12.ttf /usr/share/fonts/vista/ 
cd /usr/share/fonts/vista/ 
sudo chmod 644 YaHei.consolas.1.12.ttf
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

