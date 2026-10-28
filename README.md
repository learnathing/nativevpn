# NativeVPN

Apple Silicon（M1 及後續晶片）、macOS 13 以上的原生 VPN 客戶端，提供圖形介面、選單列狀態、連線管理與診斷功能。

目前正式版為 [0.4.1](https://github.com/learnathing/nativevpn/releases/tag/v0.4.1)。此儲存庫提供安裝包與使用說明，原始碼未公開；第三方參考授權見 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

## 支援的 VPN 服務

NativeVPN 適用於 **UniVPN／SecoClient 相容的 SSL VPN 網路擴展服務（legacy 協定）**，使用帳號與密碼登入企業或校園閘道，存取其授權的內網 IPv4 服務。這類服務屬於華為 SSL VPN／SecoClient 協定系統；實測範圍是既有的一個相容閘道，不能僅憑品牌或「SSL VPN」名稱判定相容。

| 服務或能力 | 支援範圍 |
| --- | --- |
| UniVPN／SecoClient legacy SSL VPN | 已在實際閘道驗證帳密登入、取得配置、建立隧道及內網 TCP 連通 |
| 內網 IPv4 分流 | 自動採用服務端路由與 RFC1918 私有網段的交集；一般公網流量保留原出口 |
| 內網 DNS | 僅在閘道提供明確網域與私有 DNS 時設定補充解析器；實測閘道未提供網域，因此保留系統 DNS，內網名稱解析尚未實測 |
| UniVPN v1 | 僅有合成測試的認證／配置與資料通道檢查，不列為已驗證可用服務 |
| OpenVPN、WireGuard、IPsec／IKEv2、L2TP、Cisco AnyConnect、GlobalProtect | 本客戶端未實作這些協定 |

目前支援的 legacy 配置為靜態 IPv4。DHCP、IPv6 隧道、追加驗證（例如 OTP／MFA）、端點檢查及未知認證政策不在支援範圍；遇到不支援的政策會停止連線。公開 IP 的企業服務不會經過此 VPN。

若你目前使用 UniVPN 或 SecoClient，請向管理員確認服務端協定與認證方式，再以限時連線驗證自己的環境。NativeVPN 不提供 VPN 帳號或伺服器。

## 實際驗證與限制

2026-10-28 在同一台 Mac、同一個 legacy 閘道，以 0.3.3 核心完成外網 Wi-Fi 的 15 秒連線測試：內網 TCP 可達、公網 HTTPS 取樣成功、既有出口與 DNS 保留，斷線後路由與隧道介面恢復。0.4.1 沿用該連線與分流行為，更新圖示、操作介面及提示；此次發布另通過自動化測試、安裝包檢查與 Homebrew 下載校驗，未將舊版真機結果當成 0.4.1 的跨機驗收。

在原內網 Wi-Fi，NativeVPN 與官方客戶端均有內網服務不可達的觀察；切換外網 Wi-Fi 後上述測試成功。實際可達性仍取決於網路環境及服務端授權。休眠喚醒、自動換網路恢復、長時間連線、其他閘道及第二台 Mac 的安裝／升級／卸載尚未完成驗收。

**簽章狀態：app 與背景服務採 ad-hoc 簽章，pkg 未簽署、未經 Apple 公證。** 正式版標識不代表已取得 Apple 認證；macOS 可能阻擋安裝或啟動，Homebrew 不會繞過 Gatekeeper。若組織政策要求 Developer ID 與公證，請等待符合該要求的版本。

## 安裝與更新

下載 [Releases](https://github.com/learnathing/nativevpn/releases/latest) 的 `NativeVPN-版本-arm64.pkg`，用同頁的 SHA256SUMS 核對檔案；先斷線並退出舊版 NativeVPN，結束 vpnctl，再開啟安裝包並完成管理員授權。必須安裝完整套件，app 與權限服務版本須相符。

也可使用自有 Homebrew tap：

```sh
brew tap learnathing/nativevpn
brew install --cask learnathing/nativevpn/nativevpn
```

若 Homebrew 顯示 untrusted tap，先檢視此 tap 的 Cask，再以 `brew trust --cask learnathing/nativevpn/nativevpn` 信任這個套件；這只授權 Homebrew 載入 Cask，不會繞過 macOS Gatekeeper。

更新前先在 app 斷線並退出，並結束 vpnctl：

```sh
brew update
brew upgrade --cask learnathing/nativevpn/nativevpn
```

## 使用

開啟 `/Applications/NativeVPN.app`，按「新增連線」，填入連線名稱、伺服器地址與帳號；伺服器地址可省略 `https://`。輸入密碼，首次連線請透過可信管道核對閘道憑證指紋。沒有預設帳號、密碼、閘道或信任資料。

首次使用可開啟「診斷與報告 → 限時連線（15 秒）」確認內外網與斷線清理結果，再使用主畫面的連線按鈕。顯示「已連線」代表隧道已建立，仍須實際開啟所需內網服務確認權限及可達性。主畫面底部的「網路提示」可在診斷頁查看全文。

密碼可選擇保存於 macOS 鑰匙圈。安裝包不包含個人連線、帳密、信任資料或研究日誌；0.3.5 起在簽章前移除除錯符號，並檢查展開後的全部檔案。0.3.4 因含本機建置路徑已撤下。

## 移除

先在 app 斷線並退出，結束 vpnctl，再執行：

```sh
brew uninstall --cask nativevpn
```

Brew 移除 app、權限服務與套件收據；保留場景、鑰匙圈密碼及 root 保存的閘道信任，升級也保留這些資料。需要刪除保存密碼或信任時，請在卸載前使用 app 診斷頁操作。本 tap 不提供自動清除全部資料的 zap。

直接以 pkg 安裝者應保留來源 Release 所附的 `uninstall.sh`，核對 SHA256SUMS 後，在斷線並退出程式後以 `sudo sh uninstall.sh` 執行。該腳本會移除 root 保存的信任資料，保留鑰匙圈；與 Brew 的保留策略不同。

不要公開分享含帳號、閘道地址、內網配置或完整會話的日誌。
