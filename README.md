# Давайте завершим настройку сбора метрик температуры дисков.

1. Добавьте параметр textfile collector в сервис Node Exporter
Отредактируем конфигурацию сервиса:
``

sudo systemctl edit --full node-exporter.service
```
Найдите строку ExecStart и добавьте параметр для textfile collector:

```
ExecStart=/etc/prometheus/node-exporter/node_exporter \
    --collector.textfile.directory=/var/lib/node_exporter/textfile_collector
```
Полный файл должен выглядеть примерно так:
```
ini
[Unit]
Description=Node Exporter Lesson 9.4
After=network.target

[Service]
User=art
Group=art
Type=simple
ExecStart=/etc/prometheus/node-exporter/node_exporter \
    --collector.textfile.directory=/var/lib/node_exporter/textfile_collector

[Install]
WantedBy=multi-user.target
```
2. Примените изменения и перезапустите сервис
```
sudo systemctl daemon-reload
sudo systemctl restart node-exporter
```
3. Проверьте работу
Вручную запустите скрипт сбора температуры:

``
sudo /usr/local/bin/disk_temperature.sh
```
Проверьте созданный файл с метриками:

```
cat /var/lib/node_exporter/textfile_collector/disk_temperature.prom
```
Проверьте доступность метрик через API:

```
curl -s http://localhost:9100/metrics | grep node_disk_temperature_celsius
```
4. Настройте автоматическое обновление (если еще не сделано)
```
sudo crontab -e
```
Добавьте строку для запуска каждые 5 минут:
```
*/5 * * * * /usr/local/bin/disk_temperature.sh
```
5. Дополнительные проверки
Если метрики не появляются:

Убедитесь, что Node Exporter имеет права на чтение файла:

```
sudo chmod 644 /var/lib/node_exporter/textfile_collector/disk_temperature.prom
```
Проверьте логи Node Exporter:

```
journalctl -u node-exporter -f
```
Проверьте, что скрипт работает для всех дисков:

```
sudo smartctl -A /dev/sda
sudo smartctl -A /dev/sdb  # если есть другие диски
```
Теперь метрики температуры дисков должны быть доступны в Prometheus через Node Exporter. Вы можете добавить их в свои дашборды Grafana или настроить алерты.

