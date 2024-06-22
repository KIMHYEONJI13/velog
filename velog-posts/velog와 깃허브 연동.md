<h1 id="velog-깃허브-연동-방법">velog 깃허브 연동 방법</h1>
<p>참고 <a href="https://github.com/KIMHYEONJI13/velog" title="velog">velog</a></p>
<hr />
<ol>
<li>public 레포지토리 생성</li>
<li>레포지토리명 하위
.github폴더 생성 + 하위에 workflows 폴더 생성 + update_blog.yml 파일 생성</li>
</ol>
<p>update_blog.yml 파일 [깃허브 주소 수정]</p>
<pre><code>name: Update Blog Posts

on:
push:
branches:
- main  # 또는 워크플로우를 트리거하고 싶은 브랜치 이름
schedule:
- cron: '0 0 * * *'  # 매일 자정에 실행


jobs:
update_blog:
runs-on: ubuntu-latest


steps:
- name: Checkout
  uses: actions/checkout@v2

- name: Push changes
  run: |
    git config --global user.name 'github-actions[bot]'
    git config --global user.email 'github-actions[bot]@users.noreply.github.com'
    git push https://${{ secrets.GH_PAT }}@github.com/[user명 / 깃헙주소]/velog.git

- name: Set up Python
  uses: actions/setup-python@v2
  with:
    python-version: '3.x'

- name: Install dependencies
  run: |
    pip install feedparser gitpython

- name: Run script
  run: python scripts/update_blog.py
</code></pre><ol start="3">
<li>scripts 폴더 생성 + 하위 update_blog.py 파일 생성</li>
</ol>
<p>update_blog.py 파일 [velog 아이디 수정]</p>
<pre><code>import feedparser
import git
import os

# 벨로그 RSS 피드 URL
# example : rss_url = 'https://api.velog.io/rss/@hyeonji13'
rss_url = 'https://api.velog.io/rss/@[velog 아이디]'

# 깃허브 레포지토리 경로
repo_path = '.'

# 'velog-posts' 폴더 경로
posts_dir = os.path.join(repo_path, 'velog-posts')

# 'velog-posts' 폴더가 없다면 생성
if not os.path.exists(posts_dir):
    os.makedirs(posts_dir)

# 레포지토리 로드
repo = git.Repo(repo_path)

# RSS 피드 파싱
feed = feedparser.parse(rss_url)

# 각 글을 파일로 저장하고 커밋
for entry in feed.entries:
    # 파일 이름에서 유효하지 않은 문자 제거 또는 대체
    file_name = entry.title
    file_name = file_name.replace('/', '-')  # 슬래시를 대시로 대체
    file_name = file_name.replace('\\', '-')  # 백슬래시를 대시로 대체
    # 필요에 따라 추가 문자 대체
    file_name += '.md'
    file_path = os.path.join(posts_dir, file_name)

    # 파일이 이미 존재하지 않으면 생성
    if not os.path.exists(file_path):
        with open(file_path, 'w', encoding='utf-8') as file:
            file.write(entry.description)  # 글 내용을 파일에 작성

        # 깃허브 커밋
        repo.git.add(file_path)
        repo.git.commit('-m', f'Add post: {entry.title}')

# 변경 사항을 깃허브에 푸시
repo.git.push()
</code></pre><p>폴더 생성 후 git push</p>
<ol start="4">
<li>PAT 권한 받기
github 계정 - Settings - Developer Settings - Personal access tokens (classic) - Generate New Token - 'Note'에는 토큰의 이름 쓰고 repo, workflow 클릭 - Generate new token</li>
</ol>
<p>토큰 생성 후 복사 후 저장(이동하면 토큰 복사가 안됨 1번에 복사해서 저장하기)</p>
<ol start="5">
<li>신규 생성한 레포지토리에 토큰 정보 저장
신규 레포지토리 - Settings - Actions secrets and variables - Actions - New Repository Secret
Name : 원하는 이름 작성[예시) GH_PAT] / Secret : [발급받은 토큰]</li>
</ol>
<ol start="6">
<li></li>
</ol>