import random
import time
from datetime import datetime
from web3 import Web3
from eth_account.messages import encode_defunct
import requests
import json
from loguru import logger

current_datetime = datetime.now()

keys_list = []

rpc_links = {
    'zkfair': 'https://rpc.zkfair.io',
}

with open("addresses.txt", "r") as f:
    for row in f:
        private_key = row.strip()
        if private_key:
            keys_list.append(private_key)

with open("proxies.txt", "r") as proxy_file:
    proxies = [row.strip() for row in proxy_file]


def del_key(wallet):
    file_path = f"addresses.txt"

    lines = []
    with open(file_path, "r", encoding = 'utf-8') as f:
        for row in f:
            line = row.strip()
            if line:
                if line != wallet:
                    lines.append(line + "\n")

    with open(file_path, "w", encoding = 'utf-8') as f:
        f.writelines(lines)


for private_key in keys_list:

    proxy = random.choice(proxies)
    current_proxy = {'http': 'http://' + proxy, }

    request_kwargs = {"proxies": current_proxy, "timeout": 120}

    try:
        web3 = Web3(Web3.HTTPProvider(rpc_links['zkfair'], request_kwargs = request_kwargs))
        account = web3.eth.account.from_key(private_key)
        wallet = account.address

        current_date_time = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S.%fZ')
        text_origin = current_date_time + "GET/api/airdrop?address=" + wallet
        message = encode_defunct(text = text_origin)
        text_signature = web3.eth.account.sign_message(message, private_key = private_key)
        signature_value = text_signature.signature.hex()

        url = f"https://airdrop.zkfair.io/api/airdrop?address={wallet}&API-SIGNATURE={signature_value}&TIMESTAMP={current_date_time}"

        headers = {
            "authority": "airdrop.zkfair.io",
            "accept": "application/json, text/plain, */*",
            "accept-language": "ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7",
            "origin": "https://zkfair.io",
            "referer": "https://zkfair.io/",
            "sec-ch-ua": '"Not_A Brand";v="8", "Chromium";v="120", "Google Chrome";v="120"',
            "sec-ch-ua-mobile": "?0",
            "sec-ch-ua-platform": '"Windows"',
            "sec-fetch-dest": "empty",
            "sec-fetch-mode": "cors",
            "sec-fetch-site": "same-site",
            "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
        }
        result = requests.get(url = url, proxies = current_proxy, headers = headers)

        if (result.status_code != 200):
            print(f"ошибка получения данных с API. Завершаю проверку.")
            time.sleep(1)
            break

        response_data = json.loads(result.text)
        account_profit = response_data['data']['account_profit']
        index = response_data['data']['index']

        if not account_profit:
            print(f"{wallet} = no drop")

        else:
            account_profit_decimal = int(account_profit) / 10 ** 18
            print(f"{wallet} = {account_profit_decimal}")
            with open('stats.txt', 'a') as output:
                print(f"{wallet} = {account_profit_decimal}", file = output)
        del_key(private_key)
        time.sleep(1)

    except Exception as error:
        logger.error(f'False: {error}')
        continue
