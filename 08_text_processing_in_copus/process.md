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

4. tokenization
   - corpus가 이미 정제 작업이 완료된 것이므로 바로 토크나이징을 수행하면 됨

```bash
# $(pwd) nlp-study/copus/tokenized

# 한국어 mecab tokenization
cat ../corpus.shuf.test.ko | mecab -O wakati -b 99999 | python ./post_tokenize.py ../corpus.shuf.test.ko > ./corpus.shuf.test.tok.ko

# 영어 tokenization
cat ../corpus.shuf.test.en | python ./tokenizer.py | python ./post_tokenize.py ../corpus.shuf.test.en > ./corpus.shuf.test.tok.en

# 참고 : detokenize
head ./corpus.shuf.test.tok.ko | python ./detokenizer.py

```

5. tokenization 끝나고 line수가 같은지 검증

```bash
wc -l ./corpus.shuf.*.tok.en
    202418 ./corpus.shuf.test.tok.en
    1200000 ./corpus.shuf.train.tok.en
    200000 ./corpus.shuf.valid.tok.en
    1602418 total
wc -l ./corpus.shuf.*.tok.ko
    202418 ./corpus.shuf.test.tok.ko
    1200000 ./corpus.shuf.train.tok.ko
    200000 ./corpus.shuf.valid.tok.ko
    1602418 total
```

6. subword segmentation
   - 관습적으로, vocab을 20000~30000개 정도 쌓는 것이 좋음. 영어는 분할이 더 되어 있으니 symbol을 50000 주었고, 한국어는 30000만 정도 주면 vocab이 2, 3만개 정도 됨.
   - train에서 learn_bpe했으면 train에 apply해야 함. 통째로 learn_bpe하면 기타 valid, test까지 배우는 것이므로 반칙임
   - train을 통해서 생성된 merge history 기준을 valid, test에도 진행해야 함.

```bash
# 영어.
python ./subword-nmt/learn_bpe.py --input ../tokenized/corpus.shuf.train.tok.en --output bpe.en.model --symbols 50000 --verbose
# 영어 apply bpe
cat ../tokenized/corpus.shuf.train.tok.en | python ./subword-nmt/apply_bpe.py -c ./bpe.en.model > ./corpus.shuf.train.tok.bpe.en

# 한국어 learn bpe
python ./subword-nmt/learn_bpe.py --input ../tokenized/corpus.shuf.train.tok.ko --output bpe.ko.model --symbols 30000 --verbose
# 한국어 apply bpe
cat ../tokenized/corpus.shuf.train.tok.ko | python ./subword-nmt/apply_bpe.py -c ./bpe.ko.model > ./corpus.shuf.train.tok.bpe.ko

# 이후 valid, test 세트에 대해서도 동일한 조치
# 주의할 점은, learn_bpe는 train 에서만 가하고, 그 결과물로 도출된 vocab(merge history)를 valid, test에 사용해야 한다는 것이다.

```
