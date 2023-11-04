<h1 align="center">Гайд по установке ноды <a href="https://namada.world/" target="_blank">Namada</a></h1>

> [Оффициальная документация](https://docs.namada.net/)

## Установка необходимых компонентов
### Установка rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Если Rust вас уже установлен, убедитись что у вас последняя версия
```
rustup update
```
### Установка остальных компонентов
```
sudo apt-get install -y make git-core libssl-dev pkg-config libclang-12-dev build-essential protobuf-compiler
```

## Установка namada
### Сборка из исходного кода (1 вариант)
```
git clone https://github.com/anoma/namada.git
cd namada 
make install
```

### Установка готовых бинарников (2 вариант)
```
OPERATING_SYSTEM="Linux" # or "Darwin" for MacOS
latest_release_url=$(curl -s "https://api.github.com/repos/anoma/namada/releases/latest" | grep "browser_download_url" | cut -d '"' -f 4 | grep "$OPERATING_SYSTEM")
wget "$latest_release_url"
sudo cp ./namada* /usr/local/bin/
```
После установки убедитесь что версия соответсвтует необходимой
```
namada -V
```

## Установка CometBFT
```
mkdir cometbft
cd cometbft
wget https://github.com/cometbft/cometbft/releases/download/v0.37.2/cometbft_0.37.2_linux_amd64.tar.gz
tar -xvzf cometbft_0.37.2_linux_amd64.tar.gz
sudo cp cometbft /usr/local/bin/
```
После установки убедитесь что версия соответсвтует необходимой
```
cometbft version
```

## Пре-генезис
Если вы решили стать одним из валидаторов будущей сети и подать свой pre-genesis файл, тогда для его создания вам нужно выполнить следуюшее

### Создать ключ валидатора
```
export ALIAS="ИМЯ_ВАШЕГО_ВАЛИДАТОРА"
export PUBLIC_IP="IP_ ВАШЕГО_УСТРОЙСТВА"
namada client utils init-genesis-validator --alias $ALIAS \
--max-commission-rate-change 0.01 --commission-rate 0.05 \
--net-address $PUBLIC_IP:26656
```
После этой команды создадутся ключи вашего валидаторв и отоброзится подобное сообщение:
```
Pre-genesis TOML written to $HOME/.local/share/namada
```
Это сообщение покажет где сохранены ключи вашего валидатора. В данном случае это `$HOME/.local/share/namada`
Рекомендую сохранить эти файлы где нибудь в безопасном месте, так как при их утере, вы никак не сможете восстановить своего валидатор!

Для просмотра вашего файла валидатора, который необходимо будет подавать:
```
cat $HOME/.local/share/namada/pre-genesis/$ALIAS/validator.toml
```

### Запуск genesis-валидатора
Если вашего валидатора приняли в genesis, то дождитесь когда команда выпустит `CHAIN_ID`, чтобы присоединтся к сети.
#### Присоединение к сети
```
export CHAIN_ID="namada-mainnet"
namada client utils join-network \
--chain-id $CHAIN_ID --genesis-validator $ALIAS
```
После успешного присоединения запускайте ноду
```
NAMADA_LOG=info CMT_LOG_LEVEL=p2p:none,pex:error NAMADA_CMT_STDOUT=true namada node ledger run
```

Если все сделаи верно, должны увидеть подобный вывод:
```
[<timestamp>] This node is a validator ...
```

## Пост-генезис
Если сеть уже запущена или вы не попали в генезис, тогда вам нужно создать своего валидатора
Сначала нужно запустить ноду 
```
NAMADA_LOG=info CMT_LOG_LEVEL=p2p:none,pex:error NAMADA_CMT_STDOUT=true namada node ledger run
```
После того как нода синхронизировалась нужно создать ключи валидатора:
```
namada wallet address gen --alias aliace
```
Выбрать имя валидатору
```
export VALIDATOR_ALIAS="ИМЯ_ВАЛИДАТОРА"
```
Создать ключи валидатора
```
namada client init-validator \
  --alias $VALIDATOR_ALIAS \
  --account-keys aliace \
  --signing-keys aliace \
  --commission-rate 0.1 \
  --max-commission-rate-change 0.5
```
После создание ключей, перезапустите ноду, используя ту же команду, чтобы она запустилась как валидатор.

Далее вам необходимо обеспечить вашего валидатора стейком, для этого перейдите по [ссылке](https://faucet.heliax.click/) и используйте кран на ваш адрес.
Проверка баланса
```
namada client balance --owner my-validator --token NAM
```
Для стейкинга выполните команду изменив кол-во токенов `amount` на необходимое
```
namada client bond \
  --validator my-validator \
  --amount 3.3
```

Готово

При возникновении сложностей, а так же любых сложностей, вы можете обратится за помощью в [дискорд сообщестов Namada](https://discord.gg/d3AWPb9m), а так же отмечать меня (jetrix), постараюсь помочь вам.




