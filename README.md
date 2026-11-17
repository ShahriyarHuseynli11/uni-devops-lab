# uni-devops-lab

Universitetin tələbə portalının (Flask + Redis) Git, Docker və Ansible ilə paketlənməsi və serverdə avtomatik qurulması.

## Layihənin quruluşu

```
uni-devops-lab/
├── app/                 # Flask tətbiqi (app.py, requirements.txt)
├── ansible/             # inventory, group_vars, playbook, şablonlar
├── Dockerfile           # python:3.12-slim əsaslı image
├── compose.yml          # portal + Redis (named volume ilə)
└── README.md
```

## Endpoint-lər

| Yol | Təsvir |
|-----|--------|
| `/` | Portalın işlədiyini göstərir |
| `/health` | `{"status": "healthy"}` qaytarır |
| `/visits` | Redis-də ziyarət sayğacını artırır |

## Lokal işə salmaq

```bash
docker build -t student-portal:1.0 .
docker compose up -d
curl http://localhost:8080/health
curl http://localhost:8080/visits      # hər çağırışda artır
docker compose down && docker compose up -d
curl http://localhost:8080/visits      # say sıfırlanmır (named volume)
```

Port və image adı `.env` faylı ilə dəyişdirilə bilər (nümunə: `.env.example`). Defaultlar: `student-portal:1.0`, port `8080`.

## Serverdə Ansible ilə qurmaq

1. Control node-dan app server-ə şifrəsiz SSH qurun:
   ```bash
   ssh-keygen -t ed25519
   ssh-copy-id <user>@<server-ip>
   ```
2. `ansible/inventory.ini` faylında `SERVER_IP` və `ansible_user` dəyərlərini yazın.
3. Kolleksiyanı quraşdırın və playbook-u işə salın:
   ```bash
   cd ansible
   ansible-galaxy collection install -r requirements.yml
   ansible-playbook playbook.yml
   ```
4. Portal `http://<server-ip>:8080` ünvanında açılır.

Port, image adı və qovluq yolu `ansible/group_vars/all.yml` faylındakı dəyişənlərdə saxlanılır. Playbook ikinci dəfə işə salındıqda `changed=0` göstərməlidir.

## Qarşılaşdığım problem

Portal konteynerdə Redis-ə qoşula bilmirdi (`Connection refused`). Səbəb: kod `REDIS_HOST` dəyişəni olmadıqda `localhost`-a qoşulur, konteynerin daxilində isə `localhost` Redis deyil, portalın özüdür. Həll: `compose.yml`-də `REDIS_HOST: redis` verildi (Compose servisi adı ilə DNS) və portal Redis-in `healthcheck`-i keçməsini gözləyir.

## Təhlükəsizlik

Repository-də parol, token və private key yoxdur. `.env`, `*.log` və açar faylları `.gitignore` ilə kənarlaşdırılıb.
