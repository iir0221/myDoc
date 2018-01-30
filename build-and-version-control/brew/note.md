# brew link Error
```
xinyuazh-mac:angularProjects xinyuan.zhang$ brew link pcre
Linking /usr/local/Cellar/pcre/8.41... 
Error: Could not symlink bin/pcre-config
```
解决
```bash
sudo chown -R xinyuan.zhang /usr/local
sudo chown -R xinyuan.zhang /usr/local/Cellar
```