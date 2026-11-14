# typepack-cryptocurrency

암호화폐 엔티티 타입팩 — VINEYARD Type Pack (`vineyard:typepack`).

**체인 중립**입니다. 체인마다 타입을 두지 않고 모든 체인이 같은 타입을 쓰며, 어느 체인인지는
`network` 속성이 가립니다. 체인별 팩을 따로 만들면 주소에서 거래로, 거래에서 주소로 가는 피벗과
엣지 규칙이 체인마다 갈라집니다.

## 타입 (category: `cryptocurrency`)

| 타입 | label | icon | 정체성 | 설명 |
|---|---|---|---|---|
| `cryptocurrency.crypto_address` | `{address} · {network}` | wallet | `network`+`address`+`memo` | 한 네트워크 위의 주소 |
| `cryptocurrency.crypto_transaction` | `{tx_hash} · {network}` | arrow-left-right | `network`+`tx_hash` | 온체인 거래 |
| `cryptocurrency.crypto_token` | `{symbol} · {network}` | coins | `network`+`contract_address` | 토큰 계약 (ERC-20, TRC-20, SPL, jetton, XRPL 발행 통화 등) |
| `cryptocurrency.crypto_service` | `name` | shuffle | `name` | 고객 자금을 맡지 않는 서비스 (믹서, 브리지, 도박, 가맹점, DeFi) |

## 설계 메모

- **네트워크는 정체성의 일부입니다.** 같은 문자열도 네트워크가 다르면 다른 계정입니다. `1…`·`3…`
  주소는 BTC, BCH(레거시 표기), BSV에서 바이트까지 같고, 같은 `0x…`도 Ethereum과 BSC에서 다른
  잔액을 가집니다. 2017-08-01 분기 전 거래는 txid까지 같아서 거래도 네트워크로 가립니다.
- **`evm`과 `unknown`은 판정 전의 값입니다.** 출처가 네트워크를 말하지 않고 형식으로도 하나로
  정해지지 않을 때 씁니다(`0x…`면 `evm`, 그 밖은 `unknown`). 형식만으로 정해지는 값(`bc1…`,
  `ltc1…`, `T…` 34자 Base58Check, CashAddr)은 그 네트워크를 씁니다. 판정 뒤의 노드는 따로 만들고,
  판정 전 노드에서 그 노드로 `same address on`을 긋습니다.
- **정규화(`normalize`)는 식별 비교에서만 씁니다.** `address`·`contract_address`는
  `crypto_address`, `tx_hash`는 `hex` 규칙입니다. 형식을 보고 접습니다: `0x`+hex 40자, 알려진
  hrp의 bech32/bech32m(체크섬 확인), CashAddr(`bitcoincash:` 접두어 제거), TON raw는 소문자로,
  Base58은 그대로. 64자 hex 해시는 `0x` 유무와 대소문자를 가리지 않습니다. 저장과 표시는 쓴
  그대로입니다.
- **다시 인코딩해야 같아지는 값은 쓰는 쪽이 정규형으로 바꿉니다.** EVM 주소는 소문자, CashAddr는
  접두어 없이 소문자, LTC `3…`는 `M…`, TRON hex(`41…`)는 `T…`, TON 사용자용 표기는 raw `0:<hex>`,
  TON 거래 해시 base64는 hex 소문자, XRPL X-address는 classic 주소와 `memo`로 씁니다.
- **가상자산사업자는 이 팩에 없습니다.** 거래소, OTC 데스크, 수탁형 지갑, 자금을 맡는 결제대행은
  Financial 팩의 `financial.exchange`입니다. 같은 사업자를 두 타입으로 두면 영영 합쳐지지 않기
  때문입니다. `crypto_service`는 자금을 맡지 않는 서비스에만 씁니다.
- **금액은 10진 문자열**(`balance`, `fee`)입니다. 정밀도를 잃지 않지만 숫자로 정렬되지 않습니다.
- **Financial 팩의 `crypto_address`·`crypto_transaction`은 옛 타입으로 남습니다.** 두 팩을 함께
  설치해도 정규화된 이름(`category.name`)이 달라 충돌하지 않습니다. 짧은 이름 `crypto_address`는
  identifier 순서에 따라 이 팩이 가져갑니다.
- **MAJOR 버전**: 타입 제거/정체성 변경 시에만. 프로퍼티 추가와 `network` 값 추가는 minor.
  `network` 값을 더하면 업데이트하지 않은 프로젝트에서는 그 값을 쓰는 노드 생성이 거부됩니다.

## `network` 값과 CAIP-2

`network`는 읽기 쉬운 이름이고, 외부 도구와 주고받을 때는 이 표로 CAIP-2 체인 ID로 바꿉니다.
CAIP-2 값은 식별에 쓰지 않으므로 나중에 고쳐도 노드는 바뀌지 않습니다.

| `network` | CAIP-2 | 주소 정규형 |
|---|---|---|
| `bitcoin` | `bip122:000000000019d6689c085ae165831e93` | Base58 `1…`·`3…`, bech32/bech32m `bc1q…`·`bc1p…`(소문자) |
| `bitcoin_cash` | `bip122:000000000000000000651ef99cb9fcbe` (분기 뒤 첫 블록) | CashAddr `q…`·`p…`(42자), `z…`·`r…`, P2SH32(61자), 접두어 없이 소문자 |
| `bitcoin_sv` | `bsv:mainnet` (draft) | Base58 |
| `bitcoin_gold` | 미등재 | Base58, `btg1…` |
| `litecoin` | `bip122:12a765e31ffd4059bada1e25190f6e98` | `L…`, `M…`, `ltc1…`(소문자). `3…`는 `M…`으로 |
| `dogecoin` | `bip122:1a91e3dace36e2be3bf030a65679fe82` | `D…`, `9…`·`A…` |
| `dash` | 미등재 | `X…`, `7…` |
| `zcash` | 미등재 | 투명 주소 `t1…`, `t3…` |
| `monero` | `monero:418015bb9ae982a1975da7d79277c270` (draft) | Base58(Monero 알파벳) |
| `verge` | 미등재 | `D…` |
| `ethereum` | `eip155:1` | `0x` + 소문자 hex 40자 |
| `ethereum_classic` | `eip155:61` | 위와 같음 |
| `bsc` | `eip155:56` | 위와 같음 |
| `polygon` | `eip155:137` | 위와 같음 |
| `arbitrum` | `eip155:42161` | 위와 같음 |
| `optimism` | `eip155:10` | 위와 같음 |
| `base` | `eip155:8453` | 위와 같음 |
| `avalanche_c` | `eip155:43114` | 위와 같음 |
| `evm` | 없음 (체인 미정) | 위와 같음 |
| `tron` | `tron:728126428` | `T…` Base58Check 34자 |
| `solana` | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | Base58 32바이트 |
| `xrpl` | `xrpl:0` | classic `r…` |
| `ton` | `tvm:-239` | raw `0:` + 소문자 hex 64자 |
| `cardano` | 미확정 (제안 `cardano:1-764824073`) | `addr1…`·`stake1…`(소문자), Byron은 Base58 |
| `bnb_beacon` | `cosmos:Binance-Chain-Tigris` (draft) | `bnb1…`(소문자) |
| `unknown` | 없음 | 쓴 그대로 |

## 엣지 규칙

엣지에는 타입이 없고 라벨과 데이터만 있으므로, 이 팩을 쓰는 플러그인은 아래 이름과 데이터 키를
씁니다. 같은 쌍에는 엣지가 하나뿐이라, 쓰는 쪽은 기존 엣지의 라벨을 바꾸지 않고 data 키만
더합니다. 출처가 여럿인 값은 출처별 최상위 키에 둡니다(커밋은 최상위 키 단위로만 병합).

| from → to | label | data |
|---|---|---|
| address → address | `sent to` | `network`, 출처별 요약 `summary_<출처>`(`asset`, `symbol`, `count`, `total`, `first`, `last`, `window_from`, `window_to`, `complete`), 최근 20건 `recent`(`tx_hash`, `asset`, `amount`, `time`), `first_funding`(`tx_hash`, `time`, `amount`, `source`), `deployment`(`tx_hash`) |
| address → address | `deployed` | `deployment`(`tx_hash`). 그 쌍에 엣지가 없을 때만 |
| address → transaction | `input to` | `amount`, `inputs`(`index`, `amount`, `prev_txid`, `prev_vout`) |
| transaction → address | `output to` | `amount`, `outputs`(`index`, `amount`) |
| address(계약) → token | `token contract` | 없음 |
| address → `financial.exchange`·`crypto_service`·`identity.organization` | `attributed to` | `attribution_<출처>`(`label`, `label_kind`: verified, prediction, public_tag, `as_of`) |
| address(`evm`·`unknown`) → address(네트워크 확정) | `same address on` | 판정 근거. EVM 계약이면 같은 소유자라는 뜻이 아님 |
| `identity.person`·`identity.organization` → address | `listed address` | `list`, `uid`, `program`, `as_of` |
| `web.url`·`infrastructure.domain` → address | `accepts payment to` | 결제 주소를 찾은 수집기와 위치 |
| `financial.crypto_address`(옛 타입)·결제용 `web.tracking_id` → address | `same as` | 변환 근거 |

## 검증

```bash
python3 -m pip install jsonschema
python3 -c "import json, jsonschema; jsonschema.validate(json.load(open('typepacks/cryptocurrency.json')), json.load(open('../registry/schemas/typepack.schema.json')))"
```

`normalize` 키는 레지스트리 스키마에 들어간 뒤에야 CI(`.github/workflows/validate.yml`, 레지스트리
`main`의 스키마를 받음)를 통과합니다.

## 라이선스

Apache-2.0.
