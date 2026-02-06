1. どんなもの？

HPE Cray Shasta EX（CSCSのAlps）上で、仮想化（VM）なしに “IaaSっぽい使い方” を可能にする強いテナント分離（Strong tenant separation）を実装した報告です。
具体的には、CSCS（HPC提供者）とPSI（利用機関）が、ネットワーク分離＋管理操作の権限制御＋（専用）ストレージなどを組み合わせて、PSIが「自分のデータセンタの延長」として計算資源を運用できるようにします。 

Infrastructure as a Service wit…

2. 先行研究と比べてどこがすごい？

ポイントは「クラウド的IaaSを、HPCの“非仮想化・高性能前提”のまま実現」している点です。

HPCは通常 単一テナント＆性能のため VM/VPN等を避ける前提が強いのに、ここでは ネットワーク・管理面を中心に“強い分離”を作ってIaaS化しています。 

Infrastructure as a Service wit…

HPEのCSM（Kubernetes上のマイクロサービス群）とSlingshotの機能をベースに、現場要件（PSI）とベンダ（HPE）を巻き込む co-design をプロジェクトとして回し、結果として TAPMS開発などに繋がった、と明確に位置付けています。 

Infrastructure as a Service wit…

「HPC on Cloud」や「OpenStack/MAAS」系の代替案は、性能・コスト・複雑性などで高級HPCには刺さらない、という整理もしています。 

Infrastructure as a Service wit…

3. 技術や手法のキモはどこ？

大きく (A)ネットワーク分離 と (B)管理操作の“安全な自己サービス化” と (C)ストレージの割り切り がキモです。

(A) ネットワーク分離：VLAN + VRF で“物理HPCを論理的に別ネット”にする

Slingshot側で VLAN/VNI を使って分割し、CSCS側ネットワークでは VRF でPSI論理ネットを分離（PSI用VRFの中に複数VLAN）。 

Infrastructure as a Service wit…

ノードをPSIに割当/解除する際は、(1) スイッチポートのVLAN付け替え、(2) ノードごとのネット設定ファイルでIP/GW付与、を 自動化して運用可能にしています。 

Infrastructure as a Service wit…

(B) 管理：CSM直叩きを避け、MantaでRBAC付きIaaS操作に“抽象化”

CSMは本来IaaS向けではなく、PSIに直接APIを渡すのは 全体影響のリスクと 学習コストで非現実的 → CSCSが Manta（API+CLI） を作って “必要な操作だけ” できるようにした。 

Infrastructure as a Service wit…

Keycloakの RBAC（HSMグループごとのrole）で、PSIは 自分の割当ノード群だけを操作可能にする。 

Infrastructure as a Service wit…

例として、CFSセッションログ取得は本来「CSM CLI→kubectl→pod特定→logs」みたいな多段操作だが、Mantaで 1コマンド化（図3の説明）。 

Infrastructure as a Service wit…

(C) ストレージ：マルチテナンシ未成熟なので“専用Lustre”で分離

当時のShasta EXのストレージはマルチテナンシが弱く、**PSI専用のLustre（ClusterStor）**を割当。さらにSSD/HDDのpoolを分け、SSD pool上に loopback scratch を作ってIOPS/メタデータ負荷に対処。 

Infrastructure as a Service wit…

4. どうやって有効だと検証した？

この論文は「性能ベンチの数値評価」よりも、**要件を満たす運用可能性（実システム統合）を“実装と運用設計で実証”**するタイプです。

PSI要件（自組織ネットワーク統合、管理者権限、サービス統合：LDAP/DNS/NTP/Slurm/AFS等）を満たすため、ネットワーク分離＋PSI側サービス基盤（K8s/VM群）に接続し、ノード起動後にAnsibleで追加設定を当てる運用像を提示（図4）。 

Infrastructure as a Service wit…

セキュリティ面も、CSCS側は “許可された操作だけ（Manta+RBAC）”、PSI側は **“PSIのポリシで管理される共同セキュリティゾーン（VLAN/VRF/IP分割）”**と責務分離を明確化しています。 

Infrastructure as a Service wit…

（※「どのベンチで何%」のような定量結果は本文にはほぼ出ていません。）

5. 議論はある？

あります。主に「実現したが、割り切りや未解決もある」という議論です。

分離の強さと運用権限のトレードオフ：PSIに高権限が必要だが、分離が不十分な段階では合意（許可操作の明文化・監視）で凌いだ、と記述があります。 

Infrastructure as a Service wit…

OS/ディストリ制約：PSIはRHEL系運用基盤を持つが、CSMが当時SUSE中心で、PSI既存のデプロイ基盤が使えない制約がある（将来解消見込み）。 

Infrastructure as a Service wit…

ストレージは“専用割当”で解いた：これは強い分離としては合理的だが、真にマルチテナントな共有ストレージをどうするかは将来課題（ML向けSSDベースのソフトウェア定義グローバルストレージ検討）。 

Infrastructure as a Service wit…

また、プロジェクト面の教訓として ロードマップ変動への適応や早期ユーザ巻き込みを強調しています。 

Infrastructure as a Service wit…

6. 次に読むべき論文は？

この論文の「直結する枝」を優先して挙げます（参考文献から）。

TAPMS / EX上のソフトウェア定義マルチテナンシ

Duckworth et al., Software-defined Multi-tenancy on HPE Cray EX Supercomputers (CUG 2023) 

Infrastructure as a Service wit…

CSCS Alpsでの“ソフトウェア定義HPC/クラウド的クラスター”の実践

Alam et al., 2023, Versatile software-defined HPC and cloud clusters on Alps… 

Infrastructure as a Service wit…

Martinasso et al., 2024, Versatile Software-Defined Cluster for HPC Using Cloud Abstractions 

Infrastructure as a Service wit…

管理基盤（CSM）とHPC向けREST API（FirecREST）

HPE, Cray System Management Documentation 

Infrastructure as a Service wit…

Cruz et al., 2020, FirecREST: a RESTful API to HPC systems 

Infrastructure as a Service wit…

ネットワーク（Slingshot）理解

De Sensi et al., 2020, An In-Depth Analysis of the Slingshot Interconnect 

Infrastructure as a Service wit…

将来方向：OpenCHAMI（ベンダ依存を減らす管理基盤）

OpenCHAMI 

Infrastructure as a Service wit…