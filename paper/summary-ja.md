# Structural Patch Protocol：AIエージェントによる推論ベース誤パッチを防ぐ構造変更プロトコル

**ステータス:** Working Paper  
**バージョン:** 0.1  
**リポジトリ:** https://github.com/StateToolsLab/structural-patch-protocol

---

## 概要

AI支援開発が標準となりつつある現在、AIエージェントは既存コード、DBスキーマ、API、UI、CSS、設定ファイル、ドキュメント、プロンプト、生成物など、多様な成果物を高速に変更できるようになっている。これは開発速度を大きく引き上げる一方で、AIが実物を確認する前に構造を推論し、もっともらしいが誤ったパッチを当てるという失敗モードを顕在化させている。

この問題意識は、既存研究でまったく未整理というわけではない。自動プログラム修復（Automated Program Repair, APR）の分野では、テストを通過するが意味的には正しくない **plausible but incorrect patch** や **patch overfitting** が問題化されてきた。また、LLMによるコード生成研究では、リポジトリ固有の依存関係、API、設定、非コード資源を誤って参照する **code hallucination** や **Project Context Conflict** が報告されている。さらに、SWE-benchのような実GitHub issueに基づく評価では、AIが実リポジトリ上でパッチを生成し、既存コードベースを変更できるかが評価対象となっている。つまり、単体関数の生成ではなく、既存成果物をどう安全に変更するかが重要な課題になっている。

ただし、これらの議論は主に「AI生成パッチの正しさ」「テスト通過と意味的正しさの乖離」「リポジトリ文脈の誤参照」を扱うものである。本稿が特に扱うのは、それらに加えて、AI支援開発の反復過程で、効かなかった局所パッチが撤回されずに積層し、最終的に何が効いているのか分からなくなる作業品質上の問題である。したがってSPPは、既存研究で指摘されている誤パッチ・幻覚・文脈衝突の問題を、実務上の変更管理プロトコルとして扱い直すものである。

典型的には、AIエージェントは「一般的にはこのカラム名だろう」「前Sprintでこの構造が完成しているはず」「このUIならこのセレクタが効くはず」「似た箇所にも同じ修正を展開できるはず」といった推論に基づき、実在しないファイル、テーブル、カラム、API、関数、セレクタ、設定値を前提に変更を始めることがある。この問題は単なるコード生成ミスではなく、既存成果物を変更する際に、推論が実物確認より先行してしまう構造的な作業品質問題である。

また、AI支援開発、特に高速な反復を特徴とするvibe coding的なワークフローでは、効かなかった局所パッチを撤回しないまま次のパッチを重ねる問題が頻発する。CSS、JS、HTML、UI状態管理、設定ファイルなどでは、後勝ち、selector specificity、読み込み順、状態更新順序によって、後から追加したパッチが偶然効いたように見えることがある。しかし、その状態では、何が本当に効いているのか、どの記述が不要なのか、どれを削除すると壊れるのかが分かりにくくなる。これは「最終的に動いた」ことと「検証済みの修正である」ことが一致しない問題である。

本稿では、この問題に対する実務プロトコルとして **Structural Patch Protocol（SPP）** を提案する。SPPは、AIエージェントによる既存成果物の変更を、推論先行の生成ではなく、証拠先行の変更として制御するための軽量な品質管理プロトコルである。SPPは、対象固定、実物確認、最小Probe Patch、効果検証、失敗パッチの取り下げ、確認済み統合、Cleanup Gate、Stop Ruleを通じて、AIによる変更作業の品質を外部から確認可能な手順に落とし込む。

SPPの中核フローは以下の通りである。

```text
Target Lock
→ Actual Artifact First
→ Probe Patch
→ Verify
→ Failed Patch Withdrawal / Verified Merge
→ Cleanup
→ Stop
```

第一に、**Actual Artifact First Gate** により、AIエージェントは既存成果物を変更する前に、必ず実物を確認する。設計メモ、前工程の報告、命名慣習、一般的な実装パターンは証拠ではない。DBであれば実DBファイル、テーブル、カラムを確認する。APIであれば実レスポンス、schema、バージョンを確認する。UIであればDOM、selector、実表示を確認する。ファイルであれば実ファイル名と配置を確認する。SPPにおいて、変更の根拠は推論ではなく、観測された成果物である。

第二に、**Probe Patch** により、AIエージェントは最初から広範囲を置換せず、代表箇所に最小の変更を一つだけ当てる。Probe Patchは、戻せる、確認できる、説明できる大きさでなければならない。ここで重要なのは、Probe Patchは試行錯誤のレイヤーを積むためのものではなく、統合前のキャリブレーションであるという点である。

第三に、**Failed Patch Withdrawal Rule** により、Probe Patchが期待した効果を出さなかった場合、AIエージェントは次のパッチを当てる前に、その失敗パッチを取り下げるか、明示的に解決しなければならない。SPPでは「効くまでパッチを積み重ねる」ことを禁止する。積層後に成功した結果は、検証済み修正とは見なさない。

第四に、**Verified Merge** により、Probe Patchで検証済みの変換だけを、構造的に同型の対象へ展開できる。似ているだけでは不十分であり、変換前後のパターンが明確で、検証済みであり、対象範囲内に収まっている必要がある。

第五に、**Cleanup Gate** により、停止前に、古いパッチ、上書き済みパッチ、効いていないパッチ、理由不明のパッチ層を取り除く。SPPにおけるCleanupは単なる削除作業ではない。Cleanupは、現在の有効構造を再発見し、canonical formへ再構成する作業である。

本稿では、SPPの設計動機として二つの観測ケースを扱う。第一は、DBスキーマやカラム名を実物確認前に推論し、実在しない構造を前提に実装したケースである。このケースは、Actual Artifact First Gateの必要性を示す。第二は、AI支援UI開発においてCSS、JS、HTMLの局所パッチが積層し、最終的にどの宣言や関数がUIを支配しているのか分かりにくくなったケースである。このケースは、Failed Patch WithdrawalとCleanup Gateの必要性を示す。

SPPの設計は三つの差別化ポイントを持つ。第一は、**証拠先行性**である。SPPは、AIエージェントに「実物を確認してから変更する」ことを要求し、推論ベースの誤変更を防ぐ。第二は、**未検証パッチ積層の禁止**である。SPPは、高速な試行錯誤そのものを否定しないが、効かなかったパッチを残したまま次のパッチを重ねることを禁止する。第三は、**確認済み変換だけの統合**である。SPPは、局所的に検証された変更だけを構造的に同型の対象へ展開し、類似性だけに基づく一括置換を防ぐ。

SAI-Protocolとの関係では、SAIが「何を触るか」を構造住所として固定するのに対し、SPPは「固定された対象をどう安全に触るか」を定義する。SAIは対象特定のための外部アンカーであり、SPPは対象特定後の変更プロセスを制御する。したがって、SPPはSAIと組み合わせることで、AIエージェントが対象を取り違えず、かつ推論や積層パッチによって成果物を劣化させないための作業品質レイヤーとして機能する。

また、SPPは**ドメイン非依存**の設計である。コード修正、DBスキーマ参照、API接続、JSON/CSV schema、UI/DOM/CSS修正、設定ファイル、プロンプト、ドキュメント、生成物、デザインアセットなど、AIエージェントが既存成果物を変更する場面であれば同じプロトコルを適用できる。SPPはコード専用のコーディング規約ではなく、AIによる変更作業の品質管理プロトコルである。

本プロトコルが主張しないことも明示する。SPPはテスト、レビュー、バージョン管理の代替ではない。SPPは形式検証やセキュリティ保証を提供しない。SPPはプロンプトテンプレートやプロンプトエンジニアリング術ではない。SPPは、AIエージェントが既存成果物を変更する際に、対象確認、実物確認、検証、撤回、統合、クリーンアップ、停止を外部から確認可能な手順として定義するものである。

本稿の貢献は以下の三点である。第一に、AIエージェントによる推論ベース誤パッチを、Actual Artifact Violationとして定式化し、実物確認を必須ゲートとする変更管理プロトコルを提示する。第二に、効かなかったProbe Patchを撤回しないまま重ねるInvalid Patch Stackingを定式化し、Failed Patch Withdrawal Ruleを中核ルールとして提示する。第三に、Cleanupを単なる削除ではなく、現在の有効構造をcanonical formへ再統合する工程として位置づけ、AI支援開発における作業品質を担保する実用プロトコルとしてSPPを提示する。

仕様書（`spec/Structural-Patch-Protocol.md`）および初期ユースケース（`use-cases/code/AI-Code-Patch-Protocol.md`）はMITライセンスの下で公開されている。

---

## キーワード

Structural Patch Protocol、AI支援開発、AIエージェント、推論ベース誤パッチ、Actual Artifact First、Probe Patch、Failed Patch Withdrawal、Invalid Patch Stacking、Cleanup Gate、vibe coding、変更管理、品質管理プロトコル

---

## 参考文献・関連資料

- Petke, J. et al. “The Patch Overfitting Problem in Automated Program Repair.” FSE 2024.  
  https://research.monash.edu/en/publications/the-patch-overfitting-problem-in-automated-program-repair-practic/

- Zhang, Z. et al. “LLM Hallucinations in Practical Code Generation.” arXiv, 2024.  
  https://arxiv.org/html/2409.20550v1

- Jimenez, C. E. et al. “SWE-bench: Can Language Models Resolve Real-World GitHub Issues?” ICLR 2024.  
  https://arxiv.org/abs/2310.06770

- SAI-Protocol repository.  
  https://github.com/StateToolsLab/sai-protocol

---

## 正式論文について

本サマリーはWorking Paperである。正式なpreprint版はPreprints.org等への投稿後にDOIとともに公開予定である。
