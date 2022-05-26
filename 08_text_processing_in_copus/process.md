터미널에서 진행함.

1. raw data를 하나의 정제된 형식으로 뭉치고 섞기
   - 반드시 이렇게 하라는 건 아니고, 형식이 같다면 분리해서 작업하는 게 나을 수도 있음.

```bash
# $(pwd)는 nlp-study/copus
cat ./raw/*.txt > ./copus.tsv
wc -l copus.tsv # 1602418줄 나옴
shuf ./copus.tsv > corpus.shuf.tsv
```

2. train/valid/test 분리

```bash
head -n 1200000 ./corpus.shuf.tsv > corpus.shuf.train.tsv
wc -l corpus.shuf.train.tsv # 실제로 1200000줄

# train 제외 남은 402418줄에서 앞에 200000줄은 valid 용으로
tail -n 402418 ./copus.shuf.tsv | head -n 200000 > ./corpus.shuf.valid.tsv

# 마지막으로 남은 202418줄은 test용으로
tail -n 202418 ./corpus.shuf.tsv > ./corpus.shuf.test.tsv
```

3. 한국어, 영어 분리

```bash
cut -f1 ./corpus.shuf.train.tsv > ./corpus.shuf.train.ko; cut -f2 ./corpus.shuf.train.tsv > ./corpus.shuf.train.en
# 나머지 valid, test에 대해서도 각각 분리
```
