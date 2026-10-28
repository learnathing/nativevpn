# NativeVPN 測試版

Apple Silicon macOS 13 以上的原生 VPN 客戶端。此儲存庫只提供安裝包與使用說明，原始碼未公開；第三方參考授權見 THIRD_PARTY_NOTICES.md。

**這是尚未完成跨機驗收的測試版。app/helper 使用 ad-hoc 簽章，pkg 未簽署，沒有 Apple 公證。macOS 可能阻擋安裝或啟動，Homebrew 不會取消這些檢查。** 若系統政策不允許安裝，請等待完成 Developer ID 簽章及公證的版本。

## 安裝與更新

下載 Releases 的 `NativeVPN-版本-arm64.pkg`，用同頁的 SHA256SUMS 核對檔案；先斷線並退出舊版 NativeVPN，結束 vpnctl，再開啟安裝包並完成管理員授權。必須安裝完整套件，app 與權限服務版本須相符。

也可使用自有 Homebrew tap：

```sh
brew tap learnathing/nativevpn
brew install --cask learnathing/nativevpn/nativevpn
```

更新前先在 app 斷線並退出，並結束 vpnctl：

```sh
brew update
brew upgrade --cask learnathing/nativevpn/nativevpn
```

## 使用

開啟 `/Applications/NativeVPN.app`，新增場景，填入你自己的 HTTPS 閘道地址與帳號。輸入密碼，首次連線請透過可信管道核對閘道憑證指紋。沒有預設帳號、密碼、閘道或信任資料。

只有服務端下發路由與 RFC1918 私有 IPv4 範圍的交集會進入 VPN；公開 IP 的企業服務不經此 VPN。只有具明確網域與私有 DNS 的配置才建立補充解析器。未知認證政策、追加驗證及端點檢查會停止，沒有宣稱完整 UniVPN 或 IPv6 隧道相容。

先用「診斷 → 限時連線（15 秒）」確認內外網與清理結果。GUI 顯示已連線只代表隧道建立，不代表全部內網服務可達。換網路、休眠喚醒、DNS 及長時間穩定性仍待完整真機驗收。

## 移除

先在 app 斷線並退出，結束 vpnctl，再執行：

```sh
brew uninstall --cask nativevpn
```

Brew 移除 app、權限服務與套件收據；保留場景、鑰匙圈密碼及 root 保存的閘道信任，升級也保留這些資料。需要刪除保存密碼或信任時，請在卸載前使用 app 診斷頁操作。本 tap 不提供自動清除全部資料的 zap。

直接以 pkg 安裝者應保留來源 Release 所附的 `uninstall.sh`，核對 SHA256SUMS 後，在斷線並退出程式後以 `sudo sh uninstall.sh` 執行。該腳本會移除 root 保存的信任資料，保留鑰匙圈；與 Brew 的保留策略不同。

不要公開分享含帳號、閘道地址、內網配置或完整會話的日誌。
