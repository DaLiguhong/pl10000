name: Update

on:
  watch:
    types: [started]
  schedule:
    - cron: 0,30 * * * *
  workflow_dispatch:

env:
  TZ: Asia/Shanghai

jobs:
  Update:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      with:
          token: ${{ secrets.LIVELIST }}

    - name: GetTime
      run: echo "DATE=$(date +'%Y-%m-%d %H:%M:%S CST')" >> $GITHUB_ENV

    - name: Update
      run: |
        # 央视源
        rm -f CCTV.m3u && wget https://raw.githubusercontent.com/fanmingming/live/main/tv/m3u/ipv6.m3u -O CCTV.m3u
        sed -i -n '/央视频道/,+1p' CCTV.m3u
        sed -i '1i #EXTM3U' CCTV.m3u
        sed -i '/^\s*$/d' CCTV.m3u

        # 卫视源
        rm -f CNTV.m3u && touch CNTV.m3u
        wget https://raw.githubusercontent.com/fanmingming/live/main/tv/m3u/ipv6.m3u -O CNTV1.m3u && sed -i -n '/卫视频道/,+1p' CNTV1.m3u
        wget https://raw.githubusercontent.com/fanmingming/live/main/tv/m3u/ipv6.m3u -O CNTV2.m3u && sed -i -n '/数字频道/,+1p' CNTV2.m3u
        cat CNTV1.m3u >> CNTV.m3u
        cat CNTV2.m3u >> CNTV.m3u
        rm -f CNTV1.m3u CNTV2.m3u
        sed -i '1i #EXTM3U' CNTV.m3u
        sed -i '/^\s*$/d' CNTV.m3u

        # 整合源
        rm -f IPTV.m3u && touch IPTV.m3u
        cat CCTV.m3u >> IPTV.m3u
        cat CNTV.m3u >> IPTV.m3u
        sed -i '/#EXTM3U/d' IPTV.m3u
        sed -i '1i #EXTM3U' IPTV.m3u
        sed -i '/^\s*$/d' IPTV.m3u

        # 节目源
        rm -f EPG.xml && wget https://epg.112114.xyz/pp.xml -O EPG.xml
        echo "Auto Update IPTV in $DATE,基于Moexin/IPTV项目修改,国内直播源同步fanmingming/live" > README.md

    - name: Clean
      run: |
        git config --local user.email "github-actions[bot]@users.noreply.github.com"
        git config --local user.name "github-actions[bot]"
        git checkout --orphan latest_branch
        git add -A
        git commit -am "$DATE"
        git branch -D main
        git branch -m main

    - name: Push
      run: git push -f origin main
