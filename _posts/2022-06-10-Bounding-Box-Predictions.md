---
layout: post
title: Bounding Box Predictions
---

# Bounding Box Predictions

Một cách để output các bounding box chính xác hơn là sử dụng thuật toán YOLO.
Cách làm như sau chia ảnh thành các lưới (grid), về cơ bản sử dụng thuật toán Classification with localization và áp dụng cho từng ô (cell) trong grid.
Ví dụ hình ảnh bên dưới với Grid 3x3 có 9 cell
