---
title: 解决，编译libexif时报错，“找不到 t-ja.gmo”
date: 2024-07-29 16:15:40
tags: 问题
---

在ubuntu 16.04上编译libexif的时候，遇到了一个问题，报错如下：

``` shell
root@9550d5ec9f71:/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release# make
make  all-recursive
make[1]: Entering directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release'
Making all in m4m
make[2]: Entering directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/m4m'
make[2]: Nothing to be done for 'all'.
make[2]: Leaving directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/m4m'
Making all in po
make[2]: Entering directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
make libexif-12.pot-update
make[3]: Entering directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
sed -e '/^#/d' remove-potcdate.sin > t-remove-potcdate.sed
mv t-remove-potcdate.sed remove-potcdate.sed
: --default-domain=libexif-12 --directory=.. \
  --add-comments=TRANSLATORS: --keyword=_ --keyword=N_ \
  --files-from=./POTFILES.in \
  --copyright-holder='Lutz Mueller and others' \
  --msgid-bugs-address=''
test ! -f libexif-12.po || { \
  if test -f ./libexif-12.pot; then \
    sed -f remove-potcdate.sed < ./libexif-12.pot > libexif-12.1po && \
    sed -f remove-potcdate.sed < libexif-12.po > libexif-12.2po && \
    if cmp libexif-12.1po libexif-12.2po >/dev/null 2>&1; then \
      rm -f libexif-12.1po libexif-12.2po libexif-12.po; \
    else \
      rm -f libexif-12.1po libexif-12.2po ./libexif-12.pot && \
      mv libexif-12.po ./libexif-12.pot; \
    fi; \
  else \
    mv libexif-12.po ./libexif-12.pot; \
  fi; \
}
make[3]: Leaving directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
test -z "de.gmo es.gmo fr.gmo pl.gmo ru.gmo vi.gmo" || make de.gmo es.gmo fr.gmo pl.gmo ru.gmo vi.gmo
make[3]: Entering directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
make libexif-12.pot-update
make[4]: Entering directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
: --default-domain=libexif-12 --directory=.. \
  --add-comments=TRANSLATORS: --keyword=_ --keyword=N_ \
  --files-from=./POTFILES.in \
  --copyright-holder='Lutz Mueller and others' \
  --msgid-bugs-address=''
test ! -f libexif-12.po || { \
  if test -f ./libexif-12.pot; then \
    sed -f remove-potcdate.sed < ./libexif-12.pot > libexif-12.1po && \
    sed -f remove-potcdate.sed < libexif-12.po > libexif-12.2po && \
    if cmp libexif-12.1po libexif-12.2po >/dev/null 2>&1; then \
      rm -f libexif-12.1po libexif-12.2po libexif-12.po; \
    else \
      rm -f libexif-12.1po libexif-12.2po ./libexif-12.pot && \
      mv libexif-12.po ./libexif-12.pot; \
    fi; \
  else \
    mv libexif-12.po ./libexif-12.pot; \
  fi; \
}
make[4]: Leaving directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
: --update de.po libexif-12.pot
rm -f de.gmo && : -c --statistics -o de.gmo de.po
mv: cannot stat 't-de.gmo': No such file or directory
Makefile:123: recipe for target 'de.gmo' failed
make[3]: *** [de.gmo] Error 1
make[3]: Leaving directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
Makefile:147: recipe for target 'stamp-po' failed
make[2]: *** [stamp-po] Error 2
make[2]: Leaving directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release/po'
Makefile:490: recipe for target 'all-recursive' failed
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/benchmark/project/libexif-0.6.14/libexif-libexif-0_6_14-release'
Makefile:399: recipe for target 'all' failed
make: *** [all] Error 2
```

可见问题是出在找不到 t-de.gmo 这个文件上。查询时候解决方法是安装一下gettext

``` shell
apt install gettext
```
