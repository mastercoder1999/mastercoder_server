# Jellyfin
- [Jellyfin](https://github.com/jellyfin/jellyfin)
- [nginx](https://nginx.org/)
- Le nvme de 2 to est une seule partition 
## Préparation
### File structure
```
sudo mkdir -p /srv/media/{movies,tv,anime,music}
```

### Groups / Perms
```
sudo groupadd media
sudo useradd jellyfin
sudo usermod -aG media jellyfin
sudo usermod -aG media maksim
```
```
sudo chown -R :media /srv/media
sudo chmod -R 775 /srv/media
sudo chmod g+s /srv/media
```

## Install
### Add repo
```
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg

echo "deb [signed-by=/usr/share/keyrings/jellyfin.gpg] https://repo.jellyfin.org/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list

sudo apt update
```
### Install app
```
sudo apt install jellyfin -y
```

### Service
```
sudo systemctl enable jellyfin
sudo systemctl start jellyfin
```

### UFW
```
sudo ufw allow 8096/tcp
```

http://192.168.1.169:8096

## Hardware acceleration
Nvidia driver
```
sudo apt install nvidia-driver-535 -y

sudo reboot

nvidia-smi
sudo apt install ffmpeg -y
ffmpeg -encoders | grep nvenc
```
You want:

- h264_nvenc
- hevc_nvenc

```
sudo usermod -aG video jellyfin
sudo systemctl restart jellyfin

```
IMAGE OPTIONS


### Test 
```
nvidia-smi
```
Voir :

- jellyfin-ffmpeg using GPU

## UTILISATION EXTERNE
### Install Nginx

```
sudo apt install nginx -y

sudo systemctl enable nginx
sudo systemctl start nginx
```
### Reverse Proxy
```
sudo nano /etc/nginx/sites-available/jellyfin
```
```
server {
    listen 80;
    server_name jellyfin.maksexperience.xyz;

    location / {
        proxy_pass http://127.0.0.1:8096;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
```
sudo ln -s /etc/nginx/sites-available/jellyfin /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```
### UFW
```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```
### Port Foward
Activer les ports 443 et 80 sur votre routeur
```
80 → 192.168.1.169:80
443 → 192.168.1.169:443
```
## HTTPS
### Install + certbot
```
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```
Choisir :
- Email
- Yes
- Yes
- jellyfin.maksexperience.xyz (1)

### Securité
```
sudo nano /etc/nginx/sites-available/jellyfin
```
Dans server{} :
```
client_max_body_size 20M;
proxy_buffering off;
```

```
sudo systemctl reload nginx
```
