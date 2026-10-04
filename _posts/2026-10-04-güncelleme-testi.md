---
layout: post
title: Deneme
date: 2026-10-04 15:51:00 +0300
tags:
  - tag
math: true
toc: true
---

# Callout Test

Bu sayfa Jekyll callout sistemini test etmek için hazırlanmıştır.

---

## Default Blockquote

> Bu normal bir blockquote'dur.
>
> Callout değildir ve kendi varsayılan stilinde görünmelidir.

---

## NOTE

{% include callout.html
   type="note"
   title="Not"
   content="Bu bir **NOTE** callout testidir."
%}

---

## TIP

{% include callout.html
   type="tip"
   title="İpucu"
   content="Bu bir **TIP** callout testidir."
%}

---

## IMPORTANT

{% include callout.html
   type="important"
   title="Önemli"
   content="Bu bilgi **önemlidir** ve diğer callout renklerinden ayrılmalıdır."
%}

---

## WARNING

{% include callout.html
   type="warning"
   title="Dikkat"
   content="Burada dikkat edilmesi gereken bir durum var."
%}

---

## INFO

{% include callout.html
   type="info"
   title="Bilgi"
   content="Bu genel bir bilgilendirme alanıdır."
%}

---

## CAUTION

{% include callout.html
   type="caution"
   title="Uyarı"
   content="Bu daha ciddi bir uyarı alanıdır."
%}

---

## Başlıksız / Varsayılan Başlık

{% include callout.html
   type="note"
   content="Başlık belirtilmediğinde ne olduğunu test ediyoruz."
%}

---

## Özel Başlık

{% include callout.html
   type="warning"
   title="Özel Başlık"
   content="Callout türü warning, fakat başlık özel olarak değiştirildi."
%}

---

## Aynı Türden İki Callout

{% include callout.html
   type="info"
   title="Bilgi 1"
   content="İlk bilgi kutusu."
%}

{% include callout.html
   type="info"
   title="Bilgi 2"
   content="İkinci bilgi kutusu."
%}
