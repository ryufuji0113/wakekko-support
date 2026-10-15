# 本文の事実確認

確認日: 2026年10月15日。初版の確認履歴と、同日の指定文への置き換え後の再確認を記録する。現在のHTMLは今回指定された文をそのまま使用している。初版の表は当時の記述に対する履歴であり、現在の判定は末尾の「再修正後の確認（今回のA・B）」を参照。

- P: `/Users/ryuki/Developer/wakekko-pairing-link/Wakekko/Wakekko/`（`feature/pairing-link`、HEAD `f140f6f`）
- W: `/Users/ryuki/Developer/wakekko-pairing-link/Wakekko/WidgetShared/`
- X: `/Users/ryuki/Developer/wakekko-plus-entitlement/Wakekko/Wakekko/`（`feature/plus-entitlement`、HEAD `f712d19`）
- S: P の `Localizable.xcstrings`。T: X の `Localizable.xcstrings`。文字列の行はキーの開始行を示す。
- `Wakekko/…` は、特記がなければ pairing-link リポジトリ直下からの相対パス。
- 同じ確認印が複数の文に付いている場合は同じ行にまとめた。結果欄が「一致」でも、明記した条件や静的調査の限界を含む。コード・文字列の読み取りによる確認であり、実機・iCloud実環境・App Storeでの動作試験は行っていない。

## 初版の確認履歴

| 印 | 本文の主張 | 確かめた結果 | 根拠（ファイル:行、キーと ja の値） |
|---|---|---|---|
| 【確認A】 | 二人の台帳に名前・色、支出と明細、精算・借金・返済、定期支払、カテゴリ・予算、月の締め、レポートを保存する。 | 一致 | P/Models/Member.swift:10、P/Models/SharedExpense.swift:12、P/Models/ExpenseLineItem.swift:13、P/Models/Settlement.swift:8、P/Models/Debt.swift:8、P/Models/DebtPayment.swift:8、P/Models/RecurringExpense.swift:15、P/Models/LedgerCategory.swift:14、P/Models/MonthlyClosing.swift:11、P/Models/Report.swift:10。共有レコードへの変換は P/Core/CloudKit/CloudKitSyncCoordinator.swift:2021。画面名の主張なし。 |
| 【確認B】 | 自分だけの支出は支出と同じ項目を持ち、相手に届かない。 | 一致 | P/Models/PersonalExpense.swift:11（日付・店・メモ・支払者・分け方）、P/Models/PersonalExpenseLineItem.swift:12（金額・カテゴリ・明細）。P/Core/CloudKit/PersonalAccountSession.swift:223 は private database と専用ゾーンを送信境界で検証。S:3101 `personalExpense.private` → ja「自分だけ」。 |
| 【確認C】 | レシート写真をカメラ撮影または写真ライブラリから取得する。 | 一致 | P/Expense/AddExpenseView.swift:203（photosPicker と ReceiptDocumentScanner）、同:1000（VNDocumentCameraViewController）。S:3915 `addExpense.receipt.camera` → ja「カメラで撮る」、S:3983 `addExpense.receipt.photoLibrary` → ja「写真から選ぶ」。 |
| 【確認D】 | 参加者を iCloud ユーザーレコードIDで区別し、同じアカウントで iCloud の記録を復元する。 | 一致（同期済み・共有権限が有効な記録） | P/Core/CloudKit/PersonalAccountSession.swift:177 は userRecordID().recordName を取得。P/Logic/SelfMemberProvisioner.swift:101 は本人を照合し、同:132 は ownerUserRecordName を保存。P/Core/CloudKit/CloudKitSyncCoordinator.swift:100 は private/shared database を使用、P/Core/CloudKit/CloudKitShareCoordinator.swift:145 は既存共有ゾーンを再発見、P/Core/CloudKit/RecordMapping.swift:210 は個人支出を受信復元。実機での再インストール試験は未実施。画面名の主張なし。 |
| 【確認E】 | 連絡先・位置情報・ヘルスケア・IDFAを取得しない。 | 一致（静的調査の範囲） | P 全 Swift と WidgetShared 全 Swift、Info.plist、project.pbxproj を検索し、Contacts/CoreLocation/HealthKit/AdSupport/AppTrackingTransparency、CNContact/CLLocation/HKHealth/ASIdentifierManager/ATTrackingManager の利用なし。P/Info.plist:4 に該当許可なし。P/PrivacyInfo.xcprivacy:5 は Tracking=false、同:9 は収集データ型が空。不存在に単一の行番号は付けられないため検索範囲も併記。画面名の主張なし。 |
| 【確認F】 | 写真共有は初期オフ。オン時だけ共有支出の写真を送り、個人支出の写真は端末内。画面名は「写真を相手と共有」で、オン以降に記録した写真が対象。 | 不一致（画面名と対象時点） | 初期オフは P/Core/AppSettings.swift:118、共有 CKAsset は P/Core/CloudKit/CloudKitSyncCoordinator.swift:2042、個人支出の写真が送信フィールドにないことは P/Core/CloudKit/RecordMapping.swift:192 と P/Models/PersonalExpense.swift:23 で一致。ただし S:11121 `settings.receiptImageSharing` の ja は「レシート画像も相手と共有する」。P/Settings/SettingsView.swift:566 がそのキーを使用。送信時のオン判定には作成日時の制限がなく、オン以前の記録でも再送時に写真を添付し得る。「それ以降に記録した」に限定する保証はない。OSのバックアップによる端末外保存の有無はコードだけでは確認できない。 |
| 【確認G】 | ウィジェット情報を端末のアプリ共有領域に保存し、残高・名前・今月合計をホーム／ロック画面へ表示することがある。 | 一致 | P/Widget/WidgetSnapshotPublisher.swift:40、W/WidgetSnapshotFile.swift:41（App Group）、同:54（JSONを端末保存）。W/WidgetSnapshot.swift:14 は名前・金額、同:39 は月合計。Wakekko/WakekkoWidgets/WakekkoWidgets.swift:30 と同:45 はホーム用とロック用ウィジェット。画面名の主張なし（ウィジェット専用文言は Widget.xcstrings 管理）。 |
| 【確認H】 | 一人用の招待リンク／QR、使用済みは他人が使えない。指定の招待画面・表示名保存・iCloudが必要。やり直し／取消はいつでもできる。 | 不一致（「いつでも」） | P/Core/Pairing/CloudKit/PairingLinkCloudKitDatabase.swift:24 は iCloud 確認、同:83 は oneTimeURLParticipant と readWrite、同:115 は一回用URL。P/Settings/PairingLinkOwnerView.swift:35 は同じURLをQR化、同:94 は送信。P/Core/Pairing/PairingMemberNamePolicy.swift:38 は保存名が前提。S:14491 `tab.pair` → ja「ふたり」、S:10479 `settings.pairing.invite` →「相手を招待する」、S:157 `settings.pairingLink.send` →「リンクを送る」、S:174 `.qr` →「QRを表示」、S:89 `.receive` →「招待リンクを受け取った」、S:225 `.reissue` →「招待をやり直す」、S:259 `.cancel` →「招待を取り消す」、S:123 `.warning` →「一人だけが参加できる招待リンクです。つなぎたい相手にだけ送ってください。」、S:10549 `settings.pairing.nameRequired` →「先に名前を保存してください」。ただし P/Core/Pairing/PairingLinkService.swift:128 は参加済みなら joined を返し、P/Core/Pairing/PairingLinkPresentation.swift:76 は待機中のみやり直し／取消を表示、同:81 の参加済み画面は操作なし。通信・状態確認にも成功が必要。 |
| 【確認I】 | 写真共有をオフにしても相手が受信済みの写真は相手の端末に残る。 | 一致（写真保持の挙動。画面名はFの不一致を参照） | P/Core/CloudKit/CloudKitSyncCoordinator.swift:1791 は受信写真を端末保存、P/Core/CloudKit/RecordMapping.swift:423 はローカル写真参照を保持。P/Settings/SettingsView.swift:568 は設定値だけを変更し写真削除なし。S:11155 `settings.receiptSharing.retentionNote` → ja「オフにしても、これまでに相手に届いた写真は相手の端末に残ります。」。 |
| 【確認J】 | カメラはレシートと招待QR用。OCRは端末内で行い、開発者や第三者の処理サーバーへ送らない。 | 一致（共有先への写真送信はFの条件による） | P/Expense/AddExpenseView.swift:1000 は VisionKit のカメラ、P/Settings/PairingQRCodeScannerView.swift:44 はQRだけを認識。P/Receipt/ReceiptTextRecognizer.swift:41 と同:57 は Vision、同:75 はローカルCGImageを処理。P/AI/ReceiptParser.swift:5 は FoundationModels を使用。Swift検索で外部OCR送信用URLSession等なし。カメラ利用説明は InfoPlist.xcstrings:28 `NSCameraUsageDescription` → ja「レシートの撮影とペアリング用QRの読み取りにカメラを使います。」（許可文言のため Localizable.xcstrings ではない）。 |
| 【確認K】 | 通知本文を端末内で作り、Appleのプッシュで変更を受ける。名前や金額がロック画面に出る場合がある。開発者サーバーへ送らない。 | 一致（静的調査の範囲） | P/App/ShareAcceptanceAppDelegate.swift:13 はAPNs登録、P/Core/Notifications/BackgroundNotificationFetch.swift:13 はCKNotificationを受信。P/Core/Notifications/NotificationScheduling.swift:75 はローカル通知を登録、同:81 は本文。P/Logic/Notifications/NotificationMessageBuilder.swift:50 は名前・店／カテゴリ・金額を差し込む。S:1966 `notification.body.sharedExpense` → ja「%@さんが記録しました　%@ %@」。表示可否はOSの通知／プレビュー設定にも依存。独自サーバー送信コードは検出されなかったが運営環境自体はコード監査の対象外。 |
| 【確認L】 | Plus購入・復元・購入状態取得はAppleのStoreKitで行い、購入画面から機能を解放する。カード情報を開発者へ送らない。 | 一致 | X/Core/StoreKitPlusStoreClient.swift:15 はAppleの商品取得、同:26 は purchase、同:53 は Transaction.currentEntitlements、同:95 は AppStore.sync。X/Core/PlusEntitlementPolicy.swift:34 は検証済み・未取消の買い切り権利を判定、X/Core/PlusManager.swift:102 と同:129 は購入・復元。X/Settings/PlusPurchaseView.swift:84 と同:99 にボタン。T:14882 `plusPurchase.title` → ja「わけっこ Plus」、T:15120 `plusPurchase.restore` →「購入を復元」。決済情報の入力・独自送信コードなし。 |
| 【確認M】 | CSV・JSON・Markdownと診断ログを、利用者が選んだ共有先へ渡す。診断ログに表示名・状態を含む。 | 一致 | P/Settings/SettingsView.swift:633 は3形式を作成、同:636 は ShareLink。P/Settings/DiagnosticLogView.swift:60 は診断ファイルを ShareLink で共有、同:95 はファイル生成。P/Core/DiagnosticLog.swift:193 は書出日時・build・役割・表示名・有効人数・接続状態を明示。共有先を自動選択・送信する処理なし。画面名の主張なし。 |
| 【確認N】 | PayPayボタンはアプリを開くだけで、金額・相手を渡さない。金額コピーは操作時だけ。 | 一致 | P/Settle/SettlementPaymentActions.swift:61 は引数なし `paypay://` を開く。コピーは同:46 の明示アクションと同:18 のペーストボード書込みに分離。S:12084 `settle.payment.openPayPay` → ja「PayPay を開く」、S:12067 `settle.payment.copyAmount` →「金額をコピー」。 |
| 【確認O】 | 解析・広告・行動追跡の第三者SDKやATT許可要求を組み込まない。 | 一致（静的調査の範囲） | P/PrivacyInfo.xcprivacy:5 は Tracking=false、同:7 は追跡ドメイン空。P の全 Swift と WidgetShared 全 Swift に ATT/IDFA/主要解析SDK 利用なし。Wakekko/Wakekko.xcodeproj/project.pbxproj:227 でアプリの packageProductDependencies が空。ThirdParty/BudouX は日本語改行用で、解析SDKではない。実行時の通信観測は未実施。画面名の主張なし。 |
| 【確認P】 | いつでも個別削除でき、削除は相手にも同期する。「ふたり」から共有を終えると相手は過去の台帳を見られなくなる。 | 不一致（削除の制限・解除後の閲覧） | P/History/HistoryView.swift:534 は個別削除を保存し、P/Core/CloudKit/SyncReceiveApplier.swift:269 は deletedAt を反映するため削除同期は一致。ただし P/Logic/SharedExpenseMutation.swift:19 と P/Logic/LedgerEntryMutation.swift:42 は締め済み月の削除を拒否。S:2289 `monthlyClosing.close.explanation` → ja「締めた月までの記録は、追加・修正・削除ができなくなります。お金は動かず、立替残高も変わりません。あとから解除できます。」。共有終了は P/Core/CloudKit/CloudKitShareCoordinator.swift:199、同:100 は参加解除後もローカル台帳を読み取り用に保持、同:256 と同:297 で isReadOnlySnapshot=true。P/Settings/SettingsView.swift:865 と同:873 に残った記録の説明・一覧。S:9170 `settings.pair.disconnect` → ja「つながりを解除する」、S:9476 `settings.pair.sharedUnavailable.snapshotDescription` →「この端末に残っている、見るだけの記録です。」。なお S:9034 `settings.pair.confirmDisconnectOwnerMessage` →「解除すると、相手はふたりの記録を見られなくなります。記録はこの端末に残ります。」、S:9051 の参加者向け確認文も閲覧不可と説明しており、アプリ内にも文言と保持実装の食い違いがある。 |
| 【確認Q】 | 無料は履歴直近90日、古い記録は削除しない。Plusは全履歴・残高推移・過去レポート・全期間書出しが使える買い切り。 | 一致 | X/App/AppConfig.swift:14 は90日。X/Core/PlusVisibility.swift:5 は表示限定の下限（永続化や残高計算に使用しない）、同:11 はPlusなら unlimited。X/Core/PlusFeature.swift:22 の4機能が available。X/Report/ReportView.swift:38 は閲覧制限を適用。X/Export/NormalLedgerExportCard.swift:16 と同:29 はPlusの通常書出しと全期間指定、X/Export/LedgerExportPurpose.swift:50 は権利判定。復旧・解除前の保全用書出しは別経路で無料でも可能（同:16）。X/Core/StoreKitPlusStoreClient.swift:17 は nonConsumable。T:14899 `plusPurchase.once` → ja「一度の購入で、ずっと使えます」、T:38 `plusPurchase.freeWindow` →「無料では、履歴・レポート・残高の推移を直近90日分まで見られます。」、T:14916 `plusPurchase.feature.history` →「全期間の履歴とグラフ」、T:4 `plusPurchase.feature.balanceAllTime` →「全期間の立替残高の推移」、T:21 `plusPurchase.feature.reportsAllTime` →「全期間のレポート」、T:14967 `plusPurchase.feature.export` →「書き出し（CSV・Markdown・JSON）」。返金・権利取消時は X/Core/PlusEntitlementPolicy.swift:36 により対象外。 |

## 2026年10月15日・指定文への置き換え後の再確認（B）

対象コミットは引き続き pairing-link `f140f6f` と plus-entitlement `f712d19`。以下は実装と文字列の静的確認であり、既存のアプリ・worktreeは変更せず、実機試験やiCloudの解除操作は行っていない。今回の指定文には新たな不一致もあるため、指定どおり置き換えたうえで明記する。

| 件 | 置き換え後の主張 | 結果 | 根拠（ファイル:行、キーと ja の値） |
|---|---|---|---|
| A1 / F | 「ふたり」に写真共有スイッチがあり、オン中の送信支出に写真が付く。個人写真は送らない。 | 不一致（スイッチの場所）。送信条件は一致。 | P/App/RootTabView.swift:326 は content=.pairTab、同:345 は別の設定シート content=.settingsSheet。P/Settings/SettingsView.swift:340 の pairTab 分岐には写真共有なし。写真共有は同:354 の設定シート→同:367 の receiptImageSharingCard→同:566 のToggle。S:11121 `settings.receiptImageSharing` → ja「レシート画像も相手と共有する」、S:14491 `tab.pair` → ja「ふたり」。P/Core/CloudKit/CloudKitSyncCoordinator.swift:2042 は送信時にオンなら CKAsset 添付。P/Core/CloudKit/RecordMapping.swift:192 は個人支出に写真を含めない。OSバックアップの端末外保存までは確認できない。 |
| A2 / F | オンなら支出に添えたレシート写真を相手が見られる。 | 一致 | P/Settings/SettingsView.swift:566 と P/Core/CloudKit/CloudKitSyncCoordinator.swift:2042。S:11121 `settings.receiptImageSharing` → ja「レシート画像も相手と共有する」。名称の不一致は解消。 |
| A3 / H | 相手が参加する前に招待をやり直す／取り消すと、旧リンクを使えなくできる。 | 一致（通信・状態確認・保存成功時） | P/Core/Pairing/PairingLinkService.swift:128 は参加済みなら joined、同:141 はpending参加者を削除し保存、同:160 は取消完了。P/Core/Pairing/CloudKit/PairingLinkCloudKitDatabase.swift:90 はpendingのみ削除。P/Core/Pairing/PairingLinkPresentation.swift:76 は待機中に操作を表示。S:225 `settings.pairingLink.reissue` → ja「招待をやり直す」、S:242 `settings.pairingLink.reissueConfirm` → ja「前の招待リンクは使えなくなります。新しいリンクを作りますか？」、S:259 `settings.pairingLink.cancel` → ja「招待を取り消す」、S:276 `settings.pairingLink.cancelConfirm` → ja「この招待リンクを使えなくしますか？」。 |
| A4 / I | オフでも相手に届いた写真は相手の端末に残る。 | 一致 | P/Settings/SettingsView.swift:568 は設定値変更のみ、P/Core/CloudKit/RecordMapping.swift:423 は端末の写真参照を保持。S:11155 `settings.receiptSharing.retentionNote` → ja「オフにしても、これまでに相手に届いた写真は相手の端末に残ります。」。 |
| A5 / P | 締めた月は解除まで削除できず、削除は相手にも同期する。 | 一致（対象月が保護範囲から外れた場合） | P/Logic/SharedExpenseMutation.swift:19 は削除前に締めを検査。P/Logic/MonthlyClosing/MonthlyClosingService.swift:100 は最新の有効な締めを解除し、同:110 で deletedAt を設定。MonthlyClosingPolicy.swift:12 は未削除の最新の締めだけから保護範囲を再計算し、同:37 と同:59 は保護外なら許可。最新月から順に解除する仕様（同:83）。過去に別の有効な締めが残れば、その締めまでの保護は残る。S:2731 `monthlyClosing.reopen.message` → ja「%@の記録を、また追加・修正・削除できます。お金と立替残高は変わりません。」、表示はP/Settings/MonthlyClosingView.swift:56。既存テスト Wakekko/WakekkoTests/MonthlyClosingGuardIntegrationTests.swift:261 に、締め中の削除拒否→解除後に同じ削除成功の検証あり（今回未実行）。削除反映はP/Core/CloudKit/SyncReceiveApplier.swift:269。 |
| A6 / P | 「ふたり」の「つながりを解除する」で同期が終わり、両端末に見るだけの記録が残る。アプリ削除で端末記録も消せる。 | 不一致（両側が見るだけになるという断定） | 入口はP/App/RootTabView.swift:288 と同:326、P/Settings/SettingsView.swift:351 と同:1081。S:14491 `tab.pair` → ja「ふたり」、S:9170 `settings.pair.disconnect` → ja「つながりを解除する」。持ち主側はP/Core/CloudKit/CloudKitShareCoordinator.swift:219 でCKShareのみ削除、同:391 で同じ台帳のままsoloにする。持ち主の台帳を読み取り専用にする処理はない。参加者側は同:254 で共有から離脱、同:256 はsnapshot扱い、P/Settings/SettingsView.swift:1352 は別のソロ台帳へ移行し旧台帳を保存。相手との同期停止・端末記録の保持は一致するが「それぞれ見るだけ」は不一致。アプリ削除によるOS全体のデータ・バックアップ消去はコードでは確認できない。詳細はC参照。 |
| A7 | 最終更新日は2026年10月15日のまま。 | 一致（依頼指定） | privacy.html の更新日段落は変更前後とも「最終更新日: 2026年10月15日」が1件。アプリの実装による主張ではない。 |
| A8 / H | 相手が参加する前なら「招待をやり直す」「招待を取り消す」で無効化できる。 | 一致（通信・保存成功時） | A3と同じ処理・文字列。P/Core/Pairing/PairingLinkPresentation.swift:78 が待機中に reissue/cancel を提供し、同:81 の参加済みでは提供しない。 |
| A9 / F | 「ふたり」に写真共有スイッチ。オン以前の支出も編集・再保存で写真が届くことがある。初期オフで、オフ後も受信写真は残る。 | 不一致（スイッチの場所）。挙動・名称は一致。 | スイッチはA1のとおり設定シート。P/Core/CloudKit/CloudKitSyncCoordinator.swift:2042 は作成日時にかかわらず送信時のオン判定で写真を付ける。P/Core/AppSettings.swift:118 は未設定をオフ扱い。受信写真保持はA4のとおり。S:11121 `settings.receiptImageSharing` → ja「レシート画像も相手と共有する」、S:11155 `settings.receiptSharing.retentionNote` はA4の保持文。 |
| A10 / Q | Plusの4つの箇条書きが購入画面と同じ言葉。 | 一致（全4行完全一致） | X/Settings/PlusPurchaseView.swift:44 は PlusFeature.purchaseFeatures を表示。X/Core/PlusFeature.swift:10〜13 にキー、同:22 に4機能の有効化。T:14916 `plusPurchase.feature.history` → ja「全期間の履歴とグラフ」、T:4 `plusPurchase.feature.balanceAllTime` → ja「全期間の立替残高の推移」、T:21 `plusPurchase.feature.reportsAllTime` → ja「全期間のレポート」、T:14967 `plusPurchase.feature.export` → ja「書き出し（CSV・Markdown・JSON）」。 |
| A11 / P | FAQ追加：「ふたり」から解除し同期停止、両端末に見るだけの記録が残る。 | 不一致（持ち主側まで見るだけ、との断定） | A6と同じ。参加者の旧共有台帳を保持する実装と、持ち主の同じ台帳をsoloとして維持する実装を区別する必要がある。P/Core/CloudKit/CloudKitShareCoordinator.swift:391、P/Logic/PairConnectionLedgerTransition.swift:33。 |

### 初版の不一致 F・H・P が解消したか

- F: 旧名称と「オン以降に記録したものだけ」という問題は解消。ただし今回追加した「ふたりにある」は設定シートの実装と不一致。F全体としては未解消。
- H: 「いつでも」を参加前に限定したことで解消。実際の無効化は通信・状態確認・保存の成功が前提。
- P: 締め済み月の削除制限の問題は解消。同期停止と端末記録保持も一致。ただし「それぞれの端末に見るだけ」は持ち主側のsolo台帳に当てはまらず、P全体としては未解消。
- 前回報告は、参加者のスナップショット保持を両側へ広げて説明していた。持ち主側の継続利用と、参加者が自分から離脱した際の別台帳への切り替えを十分に区別できていなかったため、今回の調査で訂正する。

## 解除の確認文と処理の調査（C・アプリ変更なし）

### 押下から確認画面まで

P/App/RootTabView.swift:288〜293 の「ふたり」タブは、同:326〜329 で SettingsView(content: .pairTab) を開く。P/Settings/SettingsView.swift:351〜352 で解除行を表示し、同:1077〜1081 の押下で pairActionGate.request(action) を呼ぶ。実際の確認画面は同:266〜275 の alert。見出し・本文・破壊的ボタンのキーは同:1276〜1312 で持ち主／参加者のアクションから選択する。

| 要素 | 持ち主側 | 参加者側 | 表示しているファイル:行 |
|---|---|---|---|
| 見出し | `settings.pair.confirmDisconnectTitle.format` → ja「%@とのつながりを解除しますか？」（S:9102） | 同じ | P/Settings/SettingsView.swift:1280〜1284。%@ は同:1316〜1320 で相手の表示名を差し込む。alert本体は同:267。 |
| 本文 | `settings.pair.confirmDisconnectOwnerMessage` → ja「解除すると、相手はふたりの記録を見られなくなります。記録はこの端末に残ります。」（S:9034） | `settings.pair.confirmDisconnectParticipantMessage` → ja「解除すると、ふたりの記録はこの端末で見られなくなります。先に書き出しておくことができます。」（S:9051） | P/Settings/SettingsView.swift:1305〜1308、alert本文の表示は同:275。 |
| 実行ボタン（destructive） | `settings.pair.disconnect` → ja「つながりを解除する」（S:9170） | 同じ | P/Settings/SettingsView.swift:1295 でキー選択、同:270〜272 で表示・実行。 |
| キャンセルボタン | `history.cancel` → ja「キャンセル」（S:6076） | 同じ | P/Settings/SettingsView.swift:273。 |

補足: `settings.pair.confirmDisconnectTitle.fallback` → ja「つながりを解除しますか？」（S:9085）は pending=nil の分岐（P/Settings/SettingsView.swift:1287〜1288）。通常の持ち主／参加者の解除では、相手名入りの format キーが使用される。

### 実際の解除処理

以下は通信・CloudKit操作と端末保存が成功した経路。失敗時にはlastErrorへ記録して停止する経路があり、相手のオフライン端末の状態が押下直後に変わる保証はない。

| 操作する側 | 共有ゾーン・CKShare | 端末の記録・同期・画面 | 根拠 |
|---|---|---|---|
| 持ち主 | 自分のprivate databaseのzone-wide CKShareを削除する。台帳のspace-ゾーンや支出等のレコードを一括削除する処理はない。参加者の共有アクセスが終わる。 | 相手MemberをdeletedAtで無効化し、持ち主のprivate同期を呼ぶ。同じactiveSpaceIDと台帳を残し、共有フラグを消してsolo状態にする。持ち主の台帳を「見るだけ」に固定する処理はなく、締め等の通常制限を除き本人は継続利用できる。相手の端末を遠隔消去しない。 | P/Settings/SettingsView.swift:1329〜1338、P/Core/CloudKit/CloudKitShareCoordinator.swift:219〜242、同:376〜382、同:391〜396、同:442〜449。solo解決はP/Core/Pairing/PairConnectionState.swift:229〜232。履歴の編集経路はP/History/HistoryView.swift:431以降。 |
| 参加者 | 自分のMemberをdeletedAtで無効化してshared同期を試みた後、shared databaseのacceptedShareIDをdeleteRecordする離脱処理。持ち主のprivate側の台帳ゾーン・支出等を一括削除する処理はない。 | 共有ゾーンIDと共有同期キャッシュを消し、isReadOnlySnapshot=trueにする。SwiftDataの旧台帳本体は削除せず保持。その後、自分の既存／新規ソロ台帳へactiveSpaceIDを切り替え、shared同期の世代を停止する。通常画面は新しいソロ台帳を表示するため、旧台帳が残っていることと、通常画面ですぐ見られることは別。 | P/Settings/SettingsView.swift:1340〜1352、P/Core/CloudKit/CloudKitShareCoordinator.swift:250〜287、P/Models/SyncMetadata.swift:565〜585、P/Settings/SettingsSyncLifecycle.swift:9〜16、P/Logic/PairConnectionLedgerTransition.swift:33〜79、P/Core/CloudKit/CloudKitSyncHost.swift:337〜345。保持期待は既存テスト Wakekko/WakekkoTests/CloudKitShareCoordinatorTests.swift:264〜295、同:298〜323（今回未実行）。 |
| 持ち主による解除を受けた参加者 | 共有ゾーンが見えなくなったことを確認すると、共有ゾーンを無効化しshared同期を停止。 | 端末の台帳は消さず保持。「記録を見る」でローカルの一覧を表示する経路がある。サーバーの共有アクセス終了と、受信済み記録の閲覧不能は同じではない。 | P/Core/CloudKit/CloudKitSyncCoordinator.swift:1307〜1315、P/Models/SyncMetadata.swift:599〜607、P/Core/Pairing/PairConnectionState.swift:221〜225 と同:50〜54、P/Settings/SettingsView.swift:1129〜1133、同:861〜901。S:9442 `settings.pair.sharedUnavailable.review` → ja「記録を見る」、S:9476 `settings.pair.sharedUnavailable.snapshotDescription` → ja「この端末に残っている、見るだけの記録です。」。 |

### 確認文と実装の食い違い

1. 持ち主側の「相手はふたりの記録を見られなくなります」は断定が強く、受信済みの端末記録を残して「記録を見る」を提供する実装と不一致。「記録はこの端末に残ります」は持ち主側の保持処理と一致するが、持ち主側が閲覧専用になるとは書かれておらず、実装でも閲覧専用にはならない。
2. 参加者側の「ふたりの記録はこの端末で見られなくなります」は、自発的離脱後に別のソロ台帳へ切り替わる通常画面の説明としては整合し得る。一方、旧台帳は端末から削除されずsnapshotとして保持されるため、データ消去や将来のあらゆる閲覧経路まで含めた閲覧不能を保証する説明としては不正確。「先に書き出しておくことができます」自体に矛盾はない。
3. 前回の「両側の確認文が保持実装と食い違う」という一括の断定は、端末内保持・共有アクセス・通常画面の表示を区別していなかった。明確な不一致は持ち主確認文の相手側ローカル閲覧に関する断定。参加者確認文は、通常画面から外れることとデータ保持の説明不足として扱う。今回指定されたサイト本文の「それぞれの端末に見るだけ」は、この区別を解決していない。

## Aの文字列・構造検証

既存HTMLタグ・属性・CSSは保持し、A11のh3・pだけを追加した。置き換え前後の件数を検査し、指定の「後」と文字単位で照合した。

| 件 | 検査結果 |
|---|---|
| A1 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A2 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A3 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A4 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A5 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A6 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A7 | PASS: 変更前後とも「最終更新日: 2026年10月15日」が1件 |
| A8 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A9 | PASS: 前1件→0件、後0件→1件、指定文字列と完全一致 |
| A10 | PASS: 前の4行ブロック1件→0件、後の4行ブロック0件→1件、指定文字列と完全一致 |
| A11 | PASS: 追加前0件→追加後1件、最後のFAQの直後にh3・pを完全一致で追加 |

## HTMLの確認

3ページとも lang=ja、UTF-8、viewport、title、description、1つのh1、全3ページへの相互リンクを用意。CSSはページ内に置き、外部CSS・JS・フォント・解析タグ・ダークモード指定なし。幅は最大720px、本文行間1.85、指定の6色を使用。要点は枠付きのp、箇条書きはul、保存先はthead/tbodyのあるtable。

## 画像

GUI自動操作・シミュレータは使用していない。AppKit/WKWebView の takeSnapshot による390px・1024pxの画像化を試したが、サンドボックス内でWebKitの補助サービスへの接続が失敗し45秒でタイムアウトした。qlmanageも sandbox initialization failed で失敗。画像は撮れなかった。

## 2026年10月15日・再修正後の確認（今回のA・B）

今回のAは4件。上の初版・前回修正の表は履歴として保持し、現在のF・Pの判定はこの節を正とする。確認対象は引き続き feature/pairing-link の HEAD f140f6f。Plus対象の feature/plus-entitlement の HEAD f712d19 も維持されているが、今回Plus本文は変更していない。アプリ・各worktreeは読み取りのみ、GUI操作・実機試験・CloudKit操作は行っていない。

### Aの前後照合

| 件 | 対象 | 前後の完全一致確認 |
|---|---|---|
| 今回A1 | privacy.html・レシート写真の保存先 | 指定の前1件→0件、指定の後0件→1件。全文を文字単位で一致確認。 |
| 今回A2 | privacy.html・つながり解除後の記録 | 指定の前1件→0件、指定の後0件→1件。全文を文字単位で一致確認。 |
| 今回A3 | support.html・写真共有FAQの冒頭 | 指定の前1件→0件、指定の後0件→1件。後続の再保存・初期オフ・写真保持の文は保持。 |
| 今回A4 | support.html・解除FAQ | 指定の前1件→0件、指定の後0件→1件。全文を文字単位で一致確認。 |

既存HTMLのタグ順・属性・CSSを変更前と照合し一致。ほかの本文と最終更新日「2026年10月15日」も保持。

### B1・設定への入口

| 項目 | 確かめた結果 | 根拠（ファイル:行、キーとjaの値） |
|---|---|---|
| 入口の場所とボタン名 | 「ホーム」画面右上の歯車ボタン。見た目はアイコンだけで、文字の「設定」は表示しない。アクセシビリティ名は「設定」。 | P/Home/HomeView.swift:202 は navigationTitle("tab.home")、同:205 は topBarTrailing、同:206〜210 は gearshape のButtonと accessibilityLabel("tab.settings")。S:14474 `tab.home` → ja「ホーム」、S:14525 `tab.settings` → ja「設定」。 |
| 押したあとの経路 | 歯車を押すと設定シートを開く。 | P/App/RootTabView.swift:264 は isPresentingSettings=true、同:301 は sheet、同:345〜349 は SettingsView(content: .settingsSheet)。 |
| 開いた画面の見出し | 「じぶんの設定」。 | P/Settings/SettingsView.swift:304 は navigationTitle(navigationTitleKey)、同:502〜503 で settingsSheet時に `settings.personal.title` を選ぶ。S:8337 `settings.personal.title` → ja「じぶんの設定」。 |
| 写真共有スイッチ | この設定シートに「レシート画像も相手と共有する」を表示する。 | P/Settings/SettingsView.swift:354〜367 の settingsSheet → personalSettingsContent → receiptImageSharingCard、同:566 の Toggle。S:11121 `settings.receiptImageSharing` → ja「レシート画像も相手と共有する」。 |
| 「アプリの設定」で通じるか | 実装との整合性はあり、設定画面の呼び方として妥当。これは導線・命名からの判断で、利用者テストによる確認ではない。見た目の入口は文字付きボタンではなく「ホーム」右上の歯車、読み上げ名は「設定」、遷移先は「じぶんの設定」。 | 上記のボタン・遷移先・見出しの実装とja値。本文は指定の「アプリの設定」のままとした。 |

### B2・今回A1〜A4の主張

| 件／主張 | 結果 | 根拠（前回Cと同じコード） |
|---|---|---|
| 今回A1・A3：設定に写真共有スイッチがあり、オン中に送る共有支出に写真を添える | 一致 | 設定の場所はB1の経路。P/Core/CloudKit/CloudKitSyncCoordinator.swift:2042 は AppSettings().receiptImageSharingEnabled がオンでローカル写真参照がある場合に CKAsset を添える。送信時に判定し、作成日時で限定しない。P/Core/CloudKit/RecordMapping.swift:192 は個人支出の送信に写真を含めない。 |
| 今回A2・A4：「ふたり」の「つながりを解除する」で相手との同期を終える | 一致（解除操作・端末保存が成功した経路） | P/App/RootTabView.swift:288〜293 と同:326〜329 が「ふたり」タブ。P/Settings/SettingsView.swift:351〜352、同:1081 に解除入口。S:14491 `tab.pair` → ja「ふたり」、S:9170 `settings.pair.disconnect` → ja「つながりを解除する」。持ち主は P/Core/CloudKit/CloudKitShareCoordinator.swift:219、同:382 でCKShare削除。参加者は同:254 で離脱し、P/Core/CloudKit/CloudKitSyncHost.swift:337〜345 でshared同期を停止する。相手のオフライン端末への反映が押下直後になるとは保証しない。 |
| 今回A2・A4：招待した側は同じ台帳を一人で使い続けられる | 一致 | P/Core/CloudKit/CloudKitShareCoordinator.swift:376〜382 はzone-wide CKShareのみ削除し台帳レコードを残す。同:226〜232 は相手Memberを無効化してprivate同期、同:391〜396 は同じactiveSpaceIDを維持してsolo状態にする。P/Core/Pairing/PairConnectionState.swift:229〜232 はsoloを解決。P/History/HistoryView.swift:448〜451 に同じ支出の編集経路があり、解除で持ち主台帳を閲覧専用にする処理はない。締め等の通常制限は残る。 |
| 今回A2・A4：参加した側にも受信済み記録が残る | 一致 | 参加者の自発的離脱は P/Core/CloudKit/CloudKitShareCoordinator.swift:250〜256 で同期情報を消してsnapshot扱いにし、台帳自体は削除しない。P/Logic/PairConnectionLedgerTransition.swift:33〜79 は旧共有台帳を保持して別のソロ台帳へ切替。持ち主から解除された場合も P/Core/CloudKit/CloudKitSyncCoordinator.swift:1307〜1315 は無効化・同期停止のみでローカルsnapshotを保持する。既存テスト Wakekko/WakekkoTests/CloudKitShareCoordinatorTests.swift:288〜295 と同:321〜323 に保持の期待がある（今回未実行）。通常画面で旧台帳をすぐ閲覧できるとは本文で保証しなくなった。 |
| 今回A2・A4：解除しても相手端末の受信済み記録を消せない | 一致（解除操作について） | P/Core/CloudKit/CloudKitShareCoordinator.swift:199〜242 と同:250〜287 の解除処理は共有アクセスの終了・Member無効化・同期状態更新であり、相手端末のレコード消去を要求する処理はない。CKShare削除は同:376〜382。共有アクセス喪失を受けた側も P/Core/CloudKit/CloudKitSyncCoordinator.swift:1312〜1315 でsnapshotを保持する。解除前の個別削除同期と、解除による遠隔消去は別。 |
| 今回A2：端末記録を消すにはその端末からアプリを削除する | OS挙動の実機確認はできていない | 解除では端末内記録を消さないことは上記で確認。「その端末から」という案内は遠隔消去ではないことと整合するが、アプリ削除・バックアップ復元を含むOS管理データ全体の挙動は今回の静的コード調査の対象外。 |

### B3・FとPの解消状況

- F：解消。既に修正済みの名称・送信対象時点に加え、残っていたスイッチの場所を「アプリの設定」に修正し、実装の導線と一致した。個人写真のOSバックアップによる端末外保存までは前回同様に確認していない。
- P：解消。締め済み月の削除制限は前回修正済み。今回「両側とも見るだけ」をやめ、持ち主の継続利用・参加者の受信済み記録保持・解除による遠隔消去不可を区別し、前回Cの実装と一致した。アプリ削除時のOS挙動の実機検証は別の未確認事項として残す。
- アプリ内の解除確認文に関する前回Cの指摘は、サイト本文の修正とは別であり、アプリは変更していない。
