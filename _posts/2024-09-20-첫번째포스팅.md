---
title: Jekyll을 기반으로 Github 블로그 만들기
date: 2024-09-20 12:00:00 +09:00
categories: [과제]
tags: [Jekyll, Github]
---

## **1. Ruby 설치**
---
Jekyll을 사용하기 위해서는 먼저 [Ruby](https://rubyinstaller.org/downloads/)를 설치해야한다.

설치가 끝난 후 `ruby -v`커맨드를 입력했을 때,
Ruby의 버전이 정상적으로 출력된다면 성공적으로 설치된 것.

```bash
$ ruby -v
ruby 3.1.6p260 (2024-05-29 revision a777087be6) [x64-mingw-ucrt]
```

## **2. Jekyll 설치**
---
Jekyll 설치를 위해서 아래의 커맨드를 입력해준다. bundler도 함께 설치.

```bash
$ gem install jekyll
$ gem install bundler
```

`jekyll -v` 커맨드와 `bundler -v` 커맨드를 입력하여 잘 설치되었는지 확인.

```bash
$ jekyll -v
jekyll 4.3.4

$ bundler -v
Bundler version 2.5.19
```

## **3. Github Repository 생성**
---
Github 블로그를 만들기 위해서는 먼저 내 github에 블로그를 위한 repository를 생성해야한다.
Repository의 이름은 `{my_github_username}.github.io`의 형식으로 생성.

새로운 repository가 생성되었다면, 내 로컬 저장소로 해당 repository를 clone 한다.
내가 원하는 로컬 디렉토리의 위치에 아래의 명령어를 통해 clone한다.

```bash
$ git clone https://github.com/{my_github_username}/{my_github_username}.github.io
```

## **4. Jekyll 사이트 생성**
---
이전 과정을 통해 생성하였던 repository가 clone 되어 내 로컬 저장소에 생성되었다.
커맨드 창을 열고 해당 저장소의 디렉토리로 이동하면, 빈 repository가 생성된 것을 확인할 수 있다.

jekyll로 사이트를 생성하기 위해 `jekyll new ./`를 입력해서 프로젝트를 생성한다.
만약 디렉토리 안에 파일이 있다면, 모두 삭제하고 아래 커맨드를 실행.

```bash
$ jekyll new ./
```

다음으로 bundle을 설치하고 업데이트 해준 후, 다시 한번 `bundle install`을 수행한다.

```bash
$ bundle install
$ bundle update
$ bundle install
```

잘 설치되었는지 확인하기 위해 로컬 서버로 실행을 해본다.

```bash
$ bundle exec jekyll serve
```

잘 실행되었다면 문구 속 로컬 주소 `http://127.0.0.1:4000/`를 확인할 수 있다.
문제가 없다면 주소로 접속하였을 때 *Welcome to Jekyll!* 이라는 문구가 표시된다.

## **5. Jekyll 테마 다운로드 및 설치**
---
[jekyll](http://jekyllthemes.org/)에서 지원하는 테마 중 원하는 것을 선택한다.
테마는 *chirpy*를 선택하였다.

Download 버튼을 클릭하여 zip 파일을 다운로드.

다운로드한 압축 파일을 압축 해제하고, 폴더 안의 파일들을 모두 내 로컬 repository에 붙여넣는다.
이미 같은 이름의 파일이 존재한다는 중복 경고가 나올 때, 모두 덮어쓰기.

이제 다운로드한 테마를 설치해야하는데, 현재 로컬 repository에는 chirpy 개발자가 커스터마이징한 상태로 업로드 되어있기 때문에 초기화를 시켜야한다.
- `Gemfile.lock` 파일 삭제
- `_posts` 디렉토리 삭제
- `docs` 디렉토리 삭제
- `/.github/workflows`에서 `pages-deploy.yml` 파일을 제외한 나머지 파일 삭제

로컬 repository의 root 디렉토리로 돌아가 `bundle install`을 실행하면 테마 적용 완료.

```bash
$ bundle install
```

## **6. 블로그 설정 변경**
---
현재 블로그는 default로 설정되어 있기 때문에, 나에게 맞춰 설정을 변경해야한다.

`_config.yml` 파일을 열어서 수정 진행.

중요한 필드에 대한 설명은 아래와 같다.

|--------------------|------------------------------------|
| **lang** | 웹 페이지의 언어 설정. `ko` |
| **timezone** | 블로그의 시간대 설정. `Asia/Seoul` |
| **title** | 블로그의 메인 타이틀 |
| **tagline** | 블로그의 서브 타이틀 |
| **description** | 블로그 소개 작성란 |
| **url** | 블로그의 URL 주소. `"https://{my_github_username}.github.io"` 형식 |
| **github_username** | GitHub 사용자 이름 |
| **social_name** | 자신의 이름 또는 닉네임 |
| **social_email** | 자신의 이메일 주소 |
| **social_links** | 자신의 소셜 미디어 링크 주소 |
| **avatar** | 프로필 사진의 링크를 입력. `/assets/img/` 디렉토리 안에 이미지 저장 후 경로를 입력. |
| **paginate** | 메인 화면에서 한 페이지에 표시할 게시글 수 |

설정 완료 후 아래의 커맨드를 통해서 로컬 서버를 실행하여 테마가 적용되었는지 확인 가능.

```bash
$ bundle exec jekyll serve
```

## **7. 포스팅 작성**
---
포스팅을 작성하기 위해서 먼저 root 디렉토리에 `_post` 디렉토리를 생성해야한다.

`_post` 디렉토리 생성 후 안에 `YYYY-MM-DD-포스팅제목.md` 형식으로 마크다운 파일을 생성한다.

마크다운 파일의 최상단에 아래와 같이 코드를 작성.

```bash
---
title: 포스팅 제목
date: YYYY-MM-DD HH:MM:SS +09:00
categories: [카테고리1, 카테고리2]
tags: [태그1, 태그2, 태그3, ...]
---
내용 입력
```

주요 마크다운 기능과 사용법은 아래와 같다.

- **제목 (Headers)**

제목은 `#` 문자를 사용하여 표시할 수 있다. `#`의 개수에 따라 제목의 레벨이 결정.

```markdown
# 제목1 (H1)
## 제목2 (H2)
### 제목3 (H3)
#### 제목4 (H4)
```

- **강조 (Emphasis)**

굵게: **텍스트** 또는 __텍스트__

기울임: *텍스트* 또는 _텍스트_

굵게+기울임: ***텍스트*** 또는 ___텍스트___

```markdown
**굵게**
*기울임*
***굵게 + 기울임***
```

- **리스트 (Lists)**

순서 없는 목록 (Unordered List): `-`, `+`, `*`를 사용하여 항목을 표시.

순서 있는 목록 (Ordered List): `숫자`와 `점(.)`을 사용.

```markdown
- 항목1
- 항목2
  - 하위 항목1
  - 하위 항목2

1. 첫 번째
2. 두 번째
   1. 하위 첫 번째
   2. 하위 두 번째
```

- **링크 (Links)**

```markdown
[링크 텍스트](URL)
```

- **이미지 (Images)**

```markdown
![이미지 설명](이미지_URL)
```

- **코드 (Code)**

인라인 코드: 백틱(``)으로 감싼다.

코드 블록: 세 개의 백틱(```)을 사용하여 여러 줄의 코드를 감싼다. 언어명을 추가하면 구문 강조가 된다.

```markdown
`인라인 코드`

```bash
# 쉘 명령어
echo "Hello, World!"
```

- **인용 (Blockquotes)**

`>` 문자를 사용하여 인용구를 만든다.

```markdown
> 인용문
```

- **수평선 (Horizontal Rules)**

세 개 이상의 `-`, `*`, `_` 문자를 사용하여 수평선을 만든다.

```markdown
---
***
___
```

- **표 (Tables)**

`|`와 `-`를 사용하여 표를 만든다.

```markdown
| 헤더1      | 헤더2      | 헤더3      |
|------------|------------|------------|
| 내용1      | 내용2      | 내용3      |
| 내용4      | 내용5      | 내용6      |
```

## **8. 빌드 및 배포**
---
마지막으로 github에 파일을 업로드하고, 자동 빌드 및 배포를 하면 블로그가 생성된다.

아래의 커맨드를 입력.

```bash
$ git add -A                          // 모든 수정 파일을 추가
$ git status                          // 파일 변경사항을 확인
$ git commit -m "원하는 커밋 제목"     // 커밋을 메시지를 작성
$ git push                            // 레포지토리로 수정 사항을 업로드
```

만약 `git add` 과정에서 아래와 같은 경고가 발생한다면

```bash
warning: in the working copy of 'YYYY-MM-DD-포스팅제목.md', 
LF will be replaced by CRLF the next time Git touches it 
```

git이 파일을 업데이트 할 때, LF(Line Feed) 줄바꿈 문자가
CRLF(Carriage Return Line Feed) 줄바꿈 문자로 대체될 것임을 나타내는 경고이므로 git이 CRLF 대신 LF를 사용하도록 강제로 설정을 변경해주면 된다.

아래의 커맨드을 적용 후, 다시 `git add`를 수행.

```bash
$ git config --global core.autocrlf true 
```

위 과정을 모두 수행하면, github에서 스스로 빌드를 진행한다.

자신의 github 레포지토리 속 Actions에서 빌드 과정을 확인할 수 있다.
빌드가 진행되고 체크 표시가 뜨면 완료된 것. 빌드가 실패하면 문제 상황을 확인 후 그 부분을 수정. 

빌드 및 배포가 완료되면, 자신의 github 블로그에 접속하여 확인 가능.