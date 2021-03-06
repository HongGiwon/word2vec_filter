# Word2Vec_filter

-학습 데이터는 나무위키의 데이터 베이스를 NamuMarkAST(https://github.com/MerHS/biryo) 를 이용하여 추출해 html 태그, 특수 문자 등을 제거하여 정제함
-디시인사이드의 글을 크롤링하여 가져온 데이터 역시 학습에 사용

-데이터 전처리 과정

1. parser로 dump json -> txt
2. editplus(텍스트 에디터)로 html 태그, 리다이렉트, 상위 문서 등의 위키 문법 텍스트 제거
4. refine.java로 특수문자 제거(정규화 기반)
5. 텍스트 에디터로 한글, control 문자, 숫자, 공백을 제외한 모든 문자 제거. ([^\u0000-\u001f\u0020\u0030-\u0039\u1100-\u11ff\u3130-\u318f\ua960-\ua97f\uac00-\ud7af\ud7b0-\ud7ff])
5. 텍스트 에디터로 [0-99][.] 같은 일부 위키 문법 텍스트 추가 제거
6. 문장 부호 (. ? !)를 기반으로 문장 단위로 나눔
7. 토크나이징(konlpy-twitter)

-학습 과정
1. word2vec 모델 생성
2. 모델 학습 (나무위키 데이터, 디시인 사이드 데이터) 
-- 모든 corpus를 메모리에 올릴수 x
-> 메모리 효율적으로 하기 위해 iteration을 위한 클래스(한번에 하나의 파일을 읽어서 문장을 yield) 만듦
--디시인 사이드 데이터의 경우 욕설을 학습하기 위한 데이터임. 욕설 목록은 나무위키 한국어 욕설 항목에서 참고함
--나무위키 약 2500만줄, 디시인사이드 약 20만줄
3. 모델 저장
4. 새로운 단어 및 문장 온라인 학습 - 데이터 불균형 문제를 해결하기 위해 기존 데이터셋에서 일부를 random sampling

-학습 후
1. 욕설을 대표하는 벡터를 생성
-대표적인 욕설 "씨발", "개새끼" 이 두 단어와 유사한 단어 100개중 욕설의 의미를 가지는 단어를 찾아 대표단어로 선정.
-이 단어들의 벡터 평균을 욕설 벡터로 설정
-욕설의 의미가 아닌 'ㅋㅋㅋ', 'ㅎㅎㅎ' 와 같은 단어들을 제외함으로써 최대한 욕설의 의미를 가지는 벡터를 생성함

-필터링
1. 욕설 벡터와 input 단어 벡터의 코사인 유사도 계산
2. 값이 threshold 이상이면 필터링
3. 얼마나 욕설에 가까운지가 수치화되므로, 다른 정보들을 사용할 수 있음
4. 텍스트 감정 분석을 추가하여, threshold를 텍스트 감정 분석의 결과에 따라 변형
추가사항) 음성 감정 분석, 영상(표정) 분석, 과거 점수 반영 등이 가능할 것으로 생각함

-결과
1. 테스트 데이터의 결과는 테스트 결과.png에서 확인 가능 
