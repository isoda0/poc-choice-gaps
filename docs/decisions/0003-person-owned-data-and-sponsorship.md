# ADR 0003: 個人データとスポンサー関係を別Aggregateにする

- Status: Accepted
- Date: 2026-08-10

## Context

長期的な収益モデルとして、企業・大学等が料金を負担し、従業員・学生本人がPremiumを利用するB2B2Cが有力になった。通常のB2B SaaSのように個人データをorganization tenant配下へ置くと、監視化、退職・卒業時の所有権、複数契約、B2Cとの移行で大きな問題が生じる。

## Decision

- 個人の行動変容データは `PersonalSpace` 配下へ置き、organizationIdを持たせない。
- スポンサーとの関係は `SponsoredProgram`、`ProgramEnrollment`、`EntitlementGrant` で表す。
- 料金の負担者と機能の利用者をEntitlementで分離する。
- スポンサーへ提供する情報はReportingPolicyを満たす集計だけにする。
- ValueTargetは本人だけが管理し、スポンサーは閲覧・設定できない。

## Consequences

- 個人課金と組織提供の間で同じPersonalSpaceを継続利用できる。
- 退職・卒業後も本人の履歴を安全に保持できる。
- スポンサー管理画面と個人行動データのアクセス経路を分離できる。
- Entitlement resolution、Consent、集計パイプラインが将来必要になる。
- 通常の単純なorganizationId付きCRUDより実装境界が増えるが、監視化と大規模データ移行を避けられる。

