---
layout: post
title: "맥에서 딥러닝을 위한 환경 구축"
date: 2019-08-23 11:12:00 +0900
tags: ["deep_learning", "mac", "setup"]
categories: ["deep_learning"]
---

1. anaconda.com/distribution/ 에서 받기
2. source bash_shell 로 아나콘다 활성화 
  - 터미널에서 `(base) -> ~` 이런식으로 나오면 활성화 된거임
3. tutorial 프로젝트 생성
  ```bash
    conda create -n tutorial python=3.7 bumpy spicy matplotlib spyder pandas seaborn scikit-learn h5py
  ```
4. tutorial 프로젝트 활성화
  - 터미널에서 `(tutorial) -> ~` 이런식으로 나오면 활성화 된거임
  ```bash
    conda activate tutorial
  ```

5. Tenserflow 설치
  ```bash
  pip install tensorflow
  ```
  - GPU를 사용하려는 경우 (mac은 지원안됨)
  ```bash
  pip install tensorflow-gpu
  ```

6. keras 설치
  ```bash
  pip install keras
  ```

7. pycharm  설치
  - 그냥 jetbrain에서 받아서 설치하면됨
  - Create New Project를 선택하고 Project path를 `PycharmProjects/deeplearning`에 생성
  - Project interpreter설정할 때 Existing Interpreter를 고른후 위에서만든 Conda Environment로 설정
    - `/home/${your_anaconda_path}/envs/tutorial/bin/python`