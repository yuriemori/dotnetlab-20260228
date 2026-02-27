---
marp: true
theme: gradient
paginate: true
header: .NET lab
---

# 管理者向けGitHub Enterpriseの運用Tips紹介: 人にもAIにも優しいプラットフォームづくり

yuriemori

---

# お品書き
- GitHub EnterpriseのあるあるQ&A
- GitHub EnterpriseでのAgent/AIの管理

---
# Yurie Mori（森　友梨映）
## Job: 
- Software Solution Engineer@Microsoft Japan

## Skills/Interests:
- GitHub/Azure DevOps/Defender for Cloud/Azure PaaS
- DevOps/DevSecOps/IaC/Platform Engineering

---

# Please follow me
X（旧Twitter）とLinkedInで発信しています。ぜひフォローしてください！

<img src=images/x-qrcode.png width="200" height="200">

<img src=images/linkedin-qrcode.png width="200" height="200">

---
# GitHub EnterpriseのあるあるQ&A
---
# Organizationは単一？複数？どちらにすべき？(1/2)
- Organizationを分けた方がいいかどうかは組織内でリポジトリ/ユーザーに対して適用したいガバナンスを特定のグループで分ける必要があるかどうかに依存する。
- Organizationでのレイヤーでは配下のリポジトリ、ユーザーに対する権限を制御することができるので、ユーザーのグループやリポジトリ群に対して同じガバナンスを適用するのであれば、Organizationを分けてるメリットはあまりない。（まったく同じ設定のOrganizationが複数存在することになるため）
---
# Organizationは単一？複数？どちらにすべき？(2/2)
- Organizationを分けるメリットは、例えば部署ごとにOrganizationを分けることで、部署ごとに異なるガバナンスを適用できることや、部署ごとに管理者を設定できることなどがある。
- なので、適用させたいガバナンスの違いごとにOrganizationを分けるのが望ましい。
- 部署ごとなど、単純にユーザーを論理的にグルーピングするのであれば、Organizationを分けるのではなくTeamを活用するのが望ましい。
- 昔はOrganizationは複数作らないほうがいいというプラクティスも紹介されていたけども、現在ではその記述はなくなっている。
- 参考：[組織のベストプラクティス](https://docs.github.com/ja/enterprise-cloud@latest/organizations/collaborating-with-groups-in-organizations/best-practices-for-organizations)

---
# 複数のOrganizationを運用するときのTips
- Custom propoertyを活用する

---
# 複数のEnterpriseを運用する際の注意点
---
# Enterpriseはいくつまで作れる？
- 4つまで。同じGitHubアカウントでEnterprise ownerになっている環境が4つある状態でEnterpriseを作成しようとすると、上限に達しているというメッセージが出てくる👇

![ghe-reached-limit](images/ghe-reached-limit.png)

- この「4つ」はNon-EMUの環境もEMUの環境も両方含めた数。
--- 
# EMUとNon-EMU, データレジデンシー版の併存はできる？
- 現在提供されているのは、個人GitHubアカウントで利用を開始するGitHub Enteprise Cloudと、Entra IDなどのIdPでユーザーアカウントをプロビジョニングするEnterprise Managed Users（EMU）版、さらにEMUの場合、日本国内にデータをホスティングするデータレジデンシー版の3つのオプションがある。
- これらの環境の併存はできる。だが、IdPの連携において、**同一Entraのテナント内で、OIDC認証によるSSOの設定は1つのGitHub Enteprise環境でのみ可能**。
---
# GitHub Enterprise CloudとIdP連携
- GitHub Enterprise Cloud - Microsoft Entra IDの連携：
  - Non-EMU（個人アカウント許容）: SAML認証のみ対応
  - EMU（企業用アカウントで運用）：OIDC, SAML認証に対応
- Enable OIDCのボタンを押してEntraに認証することでEntra IDにOIDC連携用のエンタープライズアプリケーションが登録される。
![](images/ghe-enable-oidc.png)

---
# OIDC連携ができるのは1つのEnterprise環境のみ
- OIDC連携設定時に登録されるエンタープライズアプリケーションは、同一Entraテナント内の1つのみ。
- そのため、すでにOIDC連携が完了しているGitHub Enterpriseがある状態で、2つ目のEMUの環境を作ってOIDCの連携をしようとすると、すでにエンタープライズアプリケーションがあるのでエラーになってしまう。
![](images/overlapped-oidc-error.png)
---
# SAML認証は複数の設定が可能
- SAML認証の場合は、SAML連携用のエンタープライズアプリケーションを複数同一テナント内に作ることができるため、併存させることは可能。
- OIDC連携をしているGitHub Enterprise + SAML連携をしているGitHub Enterprise→⭕
- SAML連携をしているGitHub Enterprise + SAML連携をしているGitHub Enterprise→⭕
- OIDC連携をしているGitHub Enterprise + OIDC連携をしているGitHub Enterprise→❌
---
# SAML認証で併存させる場合の注意点
- SAML連携用のエンタープライズアプリケーションはそれぞれ別なので、どのGitHub Enterpriseはどのエンタープライズアプリケーションと連携しているのかを管理する必要がある。
