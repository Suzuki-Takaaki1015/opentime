graph TD
    %% スタイル定義
    classDef dir fill:#e1f5fe,stroke:#039be5,stroke-width:2px,color:#000
    classDef rule fill:#fff3e0,stroke:#fb8c00,stroke-width:2px,color:#000
    classDef flow fill:#e8f5e9,stroke:#43a047,stroke-width:2px,color:#000
    classDef highlight fill:#ffe0b2,stroke:#f57c00,stroke-width:2px,color:#000

    subgraph Directory [ディレクトリ構造]
        direction TB
        root["happy-mignon/"]:::dir
        github[".github/"]:::dir
        issue_tpl["ISSUE_TEMPLATE"]
        pr_tpl["PULL_REQUEST_TEMPLATE.md"]
        docs["/docs<br>(要件定義, 名簿, 議事録など)"]:::dir
        hardware["/hardware<br>(3DCAD, 回路図など)"]:::dir
        happy_ws["/happy_ws<br>(ロボットのソフトウェア)"]:::dir
        readme["README.md<br>(プロジェクト説明)"]
        
        root --> github
        github --> issue_tpl
        github --> pr_tpl
        root --> docs
        root --> hardware
        root --> happy_ws
        root --> readme
    end

    subgraph Rules [GitHub運用ルール]
        direction TB
        subgraph Branch_Commit [ブランチ・コミット規則]
            r1["作業はすべて main からブランチを作成<br>※使い捨てブランチ禁止"]:::rule
            r2["【命名規則】<br>feature/Issue番号-名前<br>fix/Issue番号-名前<br>hotfix/Issue番号-名前<br>docs/Issue番号-名前"]:::rule
            r3["【コミット規則】<br>先頭にIssue番号を入れる<br>例: 10-〇〇を追加"]:::rule
            r1 --> r2 --> r3
        end

        subgraph Merge [マージ規則]
            m1["作業完了後は main へ向けた PR を作成"]:::rule
            m2["PRには 1人以上のレビュアー を設定"]:::rule
            m3["レビュアーの承認 (Approve) が<br>1つ以上でマージ可能"]:::highlight
            m1 --> m2 --> m3
        end
    end

    subgraph Onboarding [新規メンバー オンボーディングフロー]
        direction LR
        step1["① Issue作成<br>(githubの使い方を共有する-名前)"]:::flow
        step2["② ブランチ作成 ＆<br>/docs 開発メンバー名簿を編集"]:::flow
        step3["③ PR作成 ＆<br>既存メンバーが確認・承認"]:::flow
        step4["④ mainへマージ<br>(名簿更新＆成功体験の獲得)"]:::highlight
        
        step1 --> step2 --> step3 --> step4
    end