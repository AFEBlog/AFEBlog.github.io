---
layout: post
title: Build a Boat for Treasure'da Daire Yapma Rehberi
date: 2025-09-19 19:35:00 +0300
tags:
  - Roblox
  - Build-a-Boat-for-Treasure
  - daire
math: true
toc: true
---

Roblox'ta **Build a Boat for Treasure** oyununda geminizi inşa ederken düzgün bir daire yapmak bazen zor olabilir. Bu rehberde size basit bir formül ve pratik yöntem göstereceğiz.

## Dairenin Formülü

Dairenin düzgün görünmesi için blok genişliğini ve offset değerini hesaplamanız gerekiyor:

$$
\text{offset} = \frac{2 - (D \cdot \tan(180 / \text{smoothness}))}{2}
$$

![Daire Hesaplama](https://afegaming.github.io/DaireHesapl%C4%B1y%C4%B1c%C4%B1/diameter.png)

- `D`: Dairenin çapı  
- `smoothness`: Dairenin kaç blokla çizileceği  

Blok genişliğini ise şöyle hesaplıyoruz:

```python
blockWidth = D * tan(180 / smoothness)
```

> İki taraftan çıkarmayı unutmayın, böylece daireniz tam ortaya oturur.

```python
sonuç = (2 - (çap * tan(180 / smoothness))) / 2
```

## Online Hesaplama

Formüllerle uğraşmak istemiyorsanız, [online hesaplama aracım](https://afegaming.github.io/DaireHesapl%C4%B1y%C4%B1c%C4%B1/) ile hızlıca daire ölçülerinizi belirleyebilirsiniz.

## Uygulama

1. Dairenizin çapını belirleyin.
    
2. Kaç blokla çizeceğinizi seçin (smoothness).
    
3. Formülleri kullanarak blok genişliğini ve offset’i hesaplayın.
    
4. Blokları iki taraftan simetrik şekilde genişletin.
    
5. Dairenizi hazır!