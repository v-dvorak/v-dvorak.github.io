---
title: "Staff Layout Analysis Using the YOLO Platform"
collection: publications
permalink: /publication/2024-staff-layout-analysis
excerpt: 'In this paper, we bring an update on the performance
of OMR layout analysis with the state-of-the-art YOLO platform.
Compared to the MeasureDetector (the main publicly available tool for layout analysis), it achieves a similar or better
accuracy across both in-domain and out-of-domain tests over
three different datasets that we harmonized, it is more than 20x
faster, and requires more than 4 times less memory.'
date: 2024-11-22
paperurl: 'https://arxiv.org/abs/2411.15741'
citation: 'Vojtěch Dvořák, Jan jr. Hajič, and Jiří Mayer. Staff Layout Analysis Using the YOLO Platform. In Jorge Calvo-Zaragoza, Alexander Pacha, and Elona Shatri, editors, <i>Proceedings of the 6th International Workshop on Reading Music Systems</i>, pages 18-22, Online, 2024.'
---

Detecting staffs, systems, and measures, collectively
known as layout analysis, matters for Optical Music Recognition
(OMR), both because most systems today expect staff-level inputs,
and because even if these are replaced by systems that can process
the whole page, the staffs and systems are useful elements of
OMR user interfaces and applications. It receives comparatively
little attention, which is justified, as it avoids many class im-
balance, small object, and object assembly phenomena, which is
what makes OMR difficult and interesting. However, the main
publicly available tool for layout analysis, the MeasureDetector,
has not been updated for several years, and off-the-shelf object
detection has progressed: not just in accuracy, but also in speed.
Therefore, in this paper, we bring an update on the performance
of OMR layout analysis with the state-of-the-art YOLO platform.
Compared to the MeasureDetector, it achieves a similar or better
accuracy across both in-domain and out-of-domain tests over
three different datasets that we harmonized, it is more than 20x
faster, and requires more than 4 times less memory