---
author: Loko
title: 항해 플러스 - 4주차 WIL
date: 2025-04-18
description:
tags: ["항해플러스", "항해99"]
lastmod: 2025-04-18
---

## 1. 내가 구현한 아키텍처의 흐름과 구조

```sh
├── application
│   ├── balance
│   │   ├── BalanceCommand.kt
│   │   ├── BalanceFacade.kt
│   │   └── BalanceResult.kt
│   ├── coupon
│   │   ├── CouponCommand.kt
│   │   ├── CouponFacade.kt
│   │   ├── CouponResult.kt
│   │   ├── CouponScheduler.kt
│   │   └── CustomerCouponResult.kt
│   ├── dataplatform
│   │   ├── DataPlatformCommand.kt
│   │   └── DataPlatformSender.kt
│   ├── order
│   │   ├── OrderCommand.kt
│   │   ├── OrderFacade.kt
│   │   └── OrderResult.kt
│   ├── payment
│   │   ├── PaymentCommand.kt
│   │   ├── PaymentCommandFactory.kt
│   │   ├── PaymentFacade.kt
│   │   └── PaymentResult.kt
│   └── product
│       ├── ProductCriteria.kt
│       ├── ProductFacade.kt
│       └── ProductResult.kt
├── domain
│   ├── balance
│   │   ├── Balance.kt
│   │   ├── BalanceChangeType.kt
│   │   ├── BalanceHistory.kt
│   │   ├── BalanceHistoryRepository.kt
│   │   ├── BalanceHistoryService.kt
│   │   ├── BalanceRepository.kt
│   │   └── BalanceService.kt
│   ├── common
│   │   └── BaseEntity.kt
│   ├── coupon
│   │   ├── Coupon.kt
│   │   ├── CouponRepository.kt
│   │   ├── CouponService.kt
│   │   ├── CustomerCoupon.kt
│   │   ├── CustomerCouponRepository.kt
│   │   ├── CustomerCouponService.kt
│   │   ├── CustomerCouponStatus.kt
│   │   ├── DiscountPolicy.kt
│   │   ├── DiscountType.kt
│   │   ├── FixedDiscountPolicy.kt
│   │   └── RateDiscountPolicy.kt
│   ├── customer
│   │   ├── Customer.kt
│   │   ├── CustomerRepository.kt
│   │   └── CustomerService.kt
│   ├── order
│   │   ├── Order.kt
│   │   ├── OrderItem.kt
│   │   ├── OrderItemInfo.kt
│   │   ├── OrderRepository.kt
│   │   ├── OrderService.kt
│   │   └── OrderStatus.kt
│   ├── payment
│   │   ├── Payment.kt
│   │   ├── PaymentRepository.kt
│   │   └── PaymentService.kt
│   └── product
│       ├── Product.kt
│       ├── ProductInfo.kt
│       ├── ProductOption.kt
│       ├── ProductOptionRepository.kt
│       ├── ProductOptionService.kt
│       ├── ProductRepository.kt
│       ├── ProductService.kt
│       ├── Statistic.kt
│       ├── StatisticRepository.kt
│       ├── StatisticService.kt
│       ├── Stock.kt
│       ├── StockRepository.kt
│       └── StockService.kt
├── infrastructure
│   ├── balance
│   │   ├── BalanceHistoryJpaRepository.kt
│   │   ├── BalanceHistoryRepositoryImpl.kt
│   │   ├── BalanceJpaRepository.kt
│   │   └── BalanceRepositoryImpl.kt
│   ├── coupon
│   │   ├── CouponJpaRepository.kt
│   │   ├── CouponRepositoryImpl.kt
│   │   ├── CustomerCouponJpaRepository.kt
│   │   └── CustomerCouponRepositoryImpl.kt
│   ├── customer
│   │   ├── CustomerJpaRepository.kt
│   │   └── CustomerRepositoryImpl.kt
│   ├── dataplatform
│   │   └── MockApiDataPlatformSender.kt
│   ├── order
│   │   ├── OrderJpaRepository.kt
│   │   └── OrderRepositoryImpl.kt
│   ├── payment
│   │   ├── PaymentJpaRepository.kt
│   │   └── PaymentRepositoryImpl.kt
│   └── product
│       ├── PopularProductRecord.kt
│       ├── ProductJpaRepository.kt
│       ├── ProductOptionJpaRepository.kt
│       ├── ProductOptionRepositoryImpl.kt
│       ├── ProductRepositoryImpl.kt
│       ├── StatisticJooqRepository.kt
│       ├── StatisticJpaRepository.kt
│       ├── StatisticRepositoryImpl.kt
│       ├── StockJpaRepository.kt
│       └── StockRepositoryImpl.kt
├── interfaces
│   ├── balance
│   │   ├── BalanceApi.kt
│   │   ├── BalanceController.kt
│   │   ├── BalanceRequest.kt
│   │   └── BalanceResponse.kt
│   ├── coupon
│   │   ├── CouponApi.kt
│   │   ├── CouponController.kt
│   │   ├── CouponRequest.kt
│   │   └── CouponResponse.kt
│   ├── order
│   │   ├── OrderApi.kt
│   │   ├── OrderController.kt
│   │   ├── OrderRequest.kt
│   │   └── OrderResponse.kt
│   ├── payment
│   │   ├── PaymentApi.kt
│   │   ├── PaymentController.kt
│   │   ├── PaymentRequest.kt
│   │   └── PaymentResponse.kt
│   └── product
│       ├── ProductApi.kt
│       ├── ProductController.kt
│       └── ProductResponse.kt
```

Interface - Application - Domain - Infrastructure 의 4 계층 아키텍처를 위와 같이 구현했다.  
이번 주차는 그 중 Infrastructure 계층을 구현하는 주차였다.  
이미 전 주에 3개 계층에 대한 구현을 마쳤다 보니, Infrastucture 계층을 구현하고 연동하는 것은 간단한 작업이었다.  
하지만 그렇다 보니 고민을 깊게 하지 않았다는 생각도 든다.  
과제 평가 이후에 Infrastucture 계층에 대한 구현을 다들 어떻게 했는지, 다른 분들의 코드를 참고하며 배워봐야겠다.

## 2. 통합 테스트 구성 방식

API 호출에 대한 E2E 검증 케이스는 이미 API E2E 테스트에서 검증하고 있었으므로, Facade 단위에서의 통합 테스트를 작성했다.

| 테스트 대상 | 테스트 시나리오 |
|-------------|-----------------|
| [`BalanceFacade`](https://github.com/nmin11/hhplus-e-commerce/pull/25/files#diff-1ddd4a79b9fde7ab8b7ab0e3443ae86342d7be7d4cdddf8c12eee240cb2ef47d) | 잔액 충전 시 사용자 잔액 증가 및 충전 이력 생성 |
|  | 사용자 현재 잔액 조회 성공 케이스 |
|  | 사용자 잔액 변경 내역 조회 성공 케이스 |
| [`CouponFacade`](https://github.com/nmin11/hhplus-e-commerce/pull/25/files#diff-ffb84d0d78cef36af18952a9d6ad96885c6e5c31e23d11af5be5d77f817257bd) | 사용자에게 쿠폰 정상 발급 케이스 |
|  | 쿠폰 수량 없으므로 발급 불가 케이스 |
|  | 발급하려는 쿠폰이 만료된 쿠폰인 케이스 |
|  | 이미 사용자가 해당 쿠폰을 발급 받은 케이스 |
| [`ProductFacade`](https://github.com/nmin11/hhplus-e-commerce/pull/25/files#diff-9b1408d5b2e5de90094345a427847db3fbfb1c016438708af7abcb63ef28a60e) | 상품 상세 조회 성공 케이스 |
|  | 3일 간 인기 상품 조회 성공 케이스 |
| [`OrderFacade`](https://github.com/nmin11/hhplus-e-commerce/pull/25/files#diff-67e2cc83f1ffa6ece6d246d8d57529b280bdf4602f15e8112745d002fa7e928e) | 주문 생성 성공 케이스 |
|  | 주문한 옵션과 상품의 ID가 일치하지 않는 케이스 |
|  | 상품 재고가 부족한 케이스 |
|  | 0개의 상품에 대해 주문한 케이스 |
| [`PaymentFacade`](https://github.com/nmin11/hhplus-e-commerce/pull/25/files#diff-e6640ba2a3f8e79612bdba4aa2e9ba062b5c00d2e6fcd5aed0b88e081150e326) | 쿠폰을 사용한 결제의 성공 케이스 |
|  | 쿠폰을 사용하지 않은 결제의 성공 케이스 |
|  | 주문이 이미 결제되어 있는 케이스 |
|  | 상품 재고가 부족한 케이스 |
|  | 쿠폰 할인을 적용하려 하는데 해당 쿠폰이 이미 사용된 케이스 |
|  | 사용자의 잔액이 부족한 케이스 |

Facade의 Use Case 로직은 사실상 서비스 로직들의 흐름이기 때문에 실패 케이스는 서비스의 단위 테스트에서 검증되어야 하겠다고 처음에는 생각했다.  
하지만 이내 생각이 바뀌었다.  
Application 계층에서 통합 테스트를 진행한다는 것은 하위 계층들과의 연동성을 검증한다는 뜻이므로, 핵심적인 예외 케이스들은 검증해야 되겠다는 생각이 들었기 때문이다.  
이에 따라 비즈니스 요건 상 핵심적인 예외 케이스들을 도출했고, 해당 케이스들에 어울리는 테스트 코드를 작성할 수 있었다.

## 3. DB 성능 문제 분석 방식

[Line 기술 블로그 - MySQL Workbench의 VISUAL EXPLAIN으로 인덱스 동작 확인하기](https://engineering.linecorp.com/ko/blog/mysql-workbench-visual-explain-index)라는 글을 읽고, MySQL Workbench의 기능을 활용해서 DB 쿼리 성능을 분석했다.  
처음 과제를 받았을 때 나 또한 다른 동료들과 같이 몇 만 건의 쿼리, 몇 십만 건의 쿼리를 실행해보고 실행 시간을 측정해야 되겠다고 생각했다.  
하지만 멘토링 때 코치님이 위의 기술 블로그를 추천해주셨고, 이내 생각이 바뀌어서 VISUAL EXPLAIN 기술을 활용해보기로 했다.  
솔직히 MySQL 같은 데이터베이스를 다루는 기술에 익숙하지도 않은 형편에 k6 같은 분석 도구를 배워서 써봐야 한다는 것도 큰 부담이었기 때문이기도 하다.  
그 결과, 과제 요건 중 하나였던 [병목 쿼리 보고서](https://github.com/nmin11/hhplus-e-commerce/wiki/%EB%B3%91%EB%AA%A9-%EC%BF%BC%EB%A6%AC-%EB%B3%B4%EA%B3%A0%EC%84%9C)를 학습 취지에 맞게 작성할 수 있었다.

## 4. 이번 과제를 통해 느낀 점

그동안 데이터베이스 활용 기술에 대해 너무 고민해보지 않았다는 사실을 깨달았다.  
JPA 같은 기술의 활용법만 얕게 파보고, 정작 그 기술들이 어떻게 작동하는지에 대해서 깊은 관심을 가지지 않았던 것이다.  
이번에 MySQL Workbench를 활용해 봤던 것처럼, 내가 작성하는 코드가 실제로 어떤 쿼리를 작성하는지, 그 방법들을 잘 알아두어야 하겠다.
