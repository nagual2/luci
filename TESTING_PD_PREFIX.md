# Тестирование PD Prefix для WireGuard в LuCI

## Коммит
- **Репозиторий**: https://github.com/nagual2/luci
- **Коммит**: `f17c541` (luci-proto-wireguard: add PD Prefix option)
- **Ветка**: `master`

## Изменения

Добавлено поле **PD Prefix** (`pd_prefix`) в настройки интерфейса WireGuard:
- **Расположение**: `protocols/luci-proto-wireguard/htdocs/luci-static/resources/protocol/wireguard.js:175-177`
- **Тип данных**: `cidr6` (IPv6 prefix)
- **Описание**: IPv6 prefix obtained via Prefix Delegation for use by clients

---

## Рекомендации по тестированию

### 1. Сборка пакета luci-proto-wireguard

```bash
# Настройка OpenWrt SDK
git clone https://github.com/openwrt/openwrt.git
cd openwrt
git checkout v23.05.3  # или ваша версия

# Добавление Feed
echo 'src-git luci https://github.com/nagual2/luci.git' >> feeds.conf.default
./scripts/feeds update luci

# Установка пакета
./scripts/feeds install luci-proto-wireguard

# Настройка и сборка
make menuconfig  # выбрать LuCI > Protocols > luci-proto-wireguard
make -j$(nproc)
```

### 2. Установка на OpenWrt устройство

```bash
# Способ 1: через opkg (после сборки)
opkg install luci-proto-wireguard_*.ipk

# Способ 2: обновление через git (на устройстве)
cd /usr/lib/luci
git init
git remote add origin https://github.com/nagual2/luci.git
git fetch origin
git checkout origin/master -- protocols/luci-proto-wireguard/
```

### 3. Проверка поля PD Prefix в LuCI

1. Откройте веб-интерфейс LuCI (http://router-ip/cgi-bin/luci)
2. Перейдите **Network → Interfaces**
3. Нажмите **Add new interface...**
4. Выберите протокол **WireGuard VPN**
5. В разделе **General Settings** должно появиться поле **PD Prefix**

### 4. Тестирование через UCI

```bash
# Создание WireGuard интерфейса с PD Prefix
uci set network.wg0=interface
uci set network.wg0.proto='wireguard'
uci set network.wg0.private_key='YOUR_PRIVATE_KEY'
uci set network.wg0.addresses='10.0.0.1/24'
uci set network.wg0.pd_prefix='fd00:aaaa::/64'

# Просмотр настроек
uci get network.wg0.pd_prefix
# Ожидаемый вывод: fd00:aaaa::/64

# Сохранение и применение
uci commit network
/etc/init.d/network reload
```

### 5. Проверка через ubus/rpcd

```bash
# Через ubus
ubus call network.interface.wg0 status

# Через rpcd
rpcjs call luci.rpc getNetworkInterfaces
```

---

## Критерии успешного теста

- [ ] Поле **PD Prefix** отображается в веб-интерфейсе LuCI
- [ ] Поле принимает корректные IPv6 префиксы (например, `fd00:aaaa::/64`)
- [ ] Поле отклоняет некорректные значения (проверка `cidr6`)
- [ ] Значение сохраняется через UCI (`uci get network.<iface>.pd_prefix`)
- [ ] Значение корректно применяется при `network reload`

---

## Troubleshooting

```bash
# Проверка загрузки протокола
logread | grep wireguard

# Проверка JavaScript в браузере
# F12 → Console → поиск "pd_prefix"

# Проверка наличия файла
ls -la /usr/lib/luci/static/resources/protocol/wireguard.js
```
