---
title: "[Git] Github Action 알아보기"
date: 2023-08-21T09:44:52+09:00
draft: false
pin: false 
summary: "Github Action을 통해 CI/CD를 해보자"
tags: [git, ci/cd]
---

> # GitHub Actions 이해

## Overview

GitHub Actions는 빌드, 테스트 및 배포 파이프라인을 자동화할 수 있는 지속적 통합 및 지속적 배포(CI/CD) 플랫폼입니다. 리포지토리에 대한 모든 pull request를 빌드 및 테스트하는 workflows를 생성하거나 merge된 pull request를 프로덕션에 배포할 수 있습니다. GitHub Actions는 DevOps를 넘어 리포지토리에서 다른 이벤트가 발생할 때 workflows를 실행할 수 있습니다. 예를 들어 누군가 저장소에 새 issue를 생성할 때마다 적절한 레이블을 자동으로 추가하는 workflows를 실행할 수 있습니다. GitHub는 Linux, Windows 및 macOS 가상 머신을 제공하여 workflows를 실행하거나 자체 데이터 센터 또는 클라우드 인프라에서 self-hosted runners를 호스팅할 수 있습니다.

## The components of GitHub Actions

pull request가 열리거나 issue가 생성되는 등 리포지토리에서 이벤트가 발생할 때 트리거되도록 GitHub Actions _workflow_를 구성할 수 있습니다. workflows에는 순차적으로 또는 병렬로 실행할 수 있는 하나 이상의 job이 포함되어 있습니다. 각 job은 virtual machine _runner_ 또는 컨테이너 내부에서 실행되며 정의한 스크립트를 실행하거나 workflows를 단순화할 수 있는 재사용 가능한 extension인 _action_을 실행하는 하나 이상의 steps가 있습니다.



![](/images/github-action-immesion/components.png)

### Workflows

workflow**는 하나 이상의 jobs을 실행하는 구성 가능한 자동화 프로세스입니다.** workflow**는** repository**에 체크인한 YAML 파일에 의해 정의되며** repository**의 이벤트에 의해 트리거될 때 실행되거나 수동으로 또는 정의된 일정에 따라 트리거될 수 있습니다.**

Workflows**는** repository**의** `.github/workflows` **디렉터리에 정의되며,** repository**에는 각각 다른 task set을 수행할 수 있는 여러** workflows**가 있을 수 있습니다. 예를 들어** pull requests**를 작성하고 테스트하는 하나의** workflow**, release가 생성될 때마다 애플리케이션을 배포하는 또 다른 workflow, 누군가 새 issue를 열 때마다 label을 추가하는 또 다른 workflow를 가질 수 있습니다.**

**다른 workflow 내에서 다른 workflow를 참조할 수 있습니다. 자세한 내용은** "[Reusing workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)."**을 참조하십시오.**

workflows**에 대한 자세한 내용은** "[Using workflows](https://docs.github.com/en/actions/using-workflows)."**를 참조하십시오.**

### Events

event는 workflow run을 트리거하는 repository의 특정 활동입니다. 예를 들어 누군가 pull request을 생성하거나 issue를 open하거나 repository에 commit을 push할 때 활동이 GitHub에서 시작될 수 있습니다. [REST API에 post](https://docs.github.com/en/rest/repos#create-a-repository-dispatch-event)하거나 수동으로 일정에 따라 실행되도록 workflow를 트리거할 수도 있습니다.

worflows를 트리거하는 데 사용할 수 있는 전체 event list는 [Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)를 참조하세요.

### Jobs

job은 동일한 runner에서 실행되는 workflow에 있는 steps의 집합입니다. 각 step는 실행될 쉘 스크립트이거나 실행될 action입니다. steps는 순서대로 실행되며 서로 의존적입니다. 각 step은 동일한 runner에서 실행되므로 한 step에서 다른 step으로 데이터를 공유할 수 있습니다. 예를 들어 애플리케이션을 빌드하는 단계 뒤에 빌드된 애플리케이션을 테스트하는 단계가 있을 수 있습니다.

job과 다른 job들의 의존성을 구성할 수 있습니다. 기본적으로 job에는 의존성이 없으며 서로 병렬로 실행됩니다. job이 다른 job에 의존되면 의존 job이 완료될 때까지 기다렸다가 실행합니다. 예를 들어 의존성이 없는 서로 다른 아키텍처에 대한 여러 빌드 job들과 이러한 job들에 종속된 패키징 job이 있을 수 있습니다. 빌드 job은 병렬로 실행되며 모두 성공적으로 완료되면 패키징 job이 실행됩니다.

작업에 대한 자세한 내용은 "[Using jobs](https://docs.github.com/en/actions/using-jobs)."을 참조하십시오.

### Actions

action은 복잡하지만 자주 반복되는 task를 수행하는 GitHub Actions platform용 커스텀 애플리케이션입니다. action을 사용하여 workflow 파일에 작성하는 반복 코드의 양을 줄이는 데 도움이 됩니다. action은 GitHub에서 git repository를 가져오거나, 빌드 환경에 대한 올바른 툴체인을 설정하거나, 클라우드 공급자에 대한 인증을 설정할 수 있습니다.

고유한 actions를 작성하거나 GitHub Marketplace의 workflows에서 사용할 actions를 찾을 수 있습니다.

자세한 내용은 "[Creating actions](https://docs.github.com/en/actions/creating-actions)."를 참조하십시오.

### Runners

runner는 트리거될 때 workflows를 실행하는 server입니다. 각 runner는 한 번에 하나의 job을 실행할 수 있습니다. GitHub는 Ubuntu Linux, Microsoft Windows 및 macOS runner를 제공하여 workflow를 실행합니다. 각 workflow run은 새로 프로비저닝된 새로운 가상 머신에서 실행됩니다. GitHub는 또한 larger configurations에서 사용할 수 있는 larger runners를 제공합니다. 자세한 내용은 "[About larger runners](https://docs.github.com/en/actions/using-github-hosted-runners/using-larger-runners)."를 참조하십시오. 다른 운영 체제가 필요하거나 특정 하드웨어 구성이 필요한 경우 자체 runner를 host할 수 있습니다. self-hosted runners에 대한 자세한 내용은 "[Hosting your own runners](https://docs.github.com/en/actions/hosting-your-own-runners)."을 참조하십시오.