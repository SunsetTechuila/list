Я вынес эти пункты в отдельный раздел, потому что, как мне кажется, на текущий момент он предствляет собой один из самых всеобъемлющих и понятных гайдов по грамотному скрытию модификаций в системе от приложений на Android. Я лично особенно заинтересован во всех обновлениях по этой части, активно отслеживаю и проверяю все методы на своем рабочем устройстве.

## Получение root-прав
Автор рекомендует использовать первый вариант ввиду его удобства, функциональности и стабильности.

### Kitsune Mask (рекомендуется)

1. Используем [Kitsune Mask (app-release.apk).](https://github.com/HuskyDG/magisk-files/releases)
2. В настройках Kitsune Mask пересобираем приложение с рандомным именем пакета, включаем Zygisk и MagiskHide (ещё опционально можно включить биометрическую аутентификацию, для вашего удобства).
3. Настраиваем скрытие:
   * настроить Hide (рекомендуется). В списке выбираем приложения, от которых нужно скрыть доступ к рут-правам (в них все модули будут размонтированы).
   * использовать SuList. Включаем одноименную настройку, а в списке выбираем приложения, которые будут иметь доступ рут-правам и смогут их запросить. **Этот вариант несовместим с многими модулями и приводит к необратимой (!) поломке системы на устройствах без доступа к /data через Recovery в случае конфликта.**
   
### KernelSU
1. Установите [KernelSU](https://kernelsu.org/) и его менеджер [любым доступным для вас способом.](https://kernelsu.org/guide/installation.html)
2. В KSU Менеджере установите модуль [Zygisk Next](https://github.com/Dr-TSNG/ZygiskNext/releases), перезагрузитесь и в меню настроек модуля (кнопка "Открыть") включите опцию "Enforce Denylist". Перезагрузите устройство.
3. Перейдите во вкладку Суперпользователь и выберите приложения, которые должны иметь доступ к рут-правам. 

P.S. Вот и первый недостаток KernelSU – вы элементарно не можете скрыть менеджер root-прав, как это работает с пересборкой пакета в Magisk. Потому, здесь у вас есть два рабочих варианта: скрыть приложение менеджера KernelSU [с помощью Hide My Applist](#hide-my-applist-для-lsposed) или удалить его из системы, по необходимости обращаясь к функционалу менеджера через CLI `ksud`. Оставлять приложение в системе, никак не скрывая его, не рекомендуется – все (даже российские) банковские приложения распознают его в системе и блокируют свой функционал.  
И в целом, из личного опыта, на моём неофициально поддерживаемом Pixel 5, KSU работал откровенно через жопу. Активация того же Denylist'a в ZygiskNext вообще систему повесила, в то время как на POCO F5 всё нормально. **Совершенно никаких преимуществ у KSU перед Kitsune Mask нет.** Поэтому, если ваше устройство официально не поддерживается – даже не вздумайте запариваться с этой херней, оно совершенно не стоит вашего времени и нервов. Да и в любом случае, советую отдать предпочтение привычному и стабильному Magisk.

## Продвинутое скрытие сторонней прошивки, модулей и разблокированного загрузчика

<details>

<summary>Зачем всё это нужно?</summary>

Для корректного, стопроцентного сокрытия любых вмешательств в систему от самых привередливых приложений.  
И здесь нужно объяснить. Таких "привередливых" конечно, очень немного. Пока что. Сейчас, подавляющему большинству приложений, достаточно скрытых root-прав в Magisk Hide, чтобы они воспринимали устройство как безопасное и не резали функционал (или вовсе не запускались, как некоторые игры).  
На деле же Hide-ом дело не ограничивается, иные следы различных вмешательств в систему и девственность устройства по-прежнему видны всем приложениям, а как только их разработчики внедрят улучшенные проверки безопасности, «всё рухнет» – стоит только обновиться. Потому, здесь мы расскажем, как заранее позаботиться о том, чтобы наши модификации было невозможно обнаружить и избавить себя от потенциальных нежданных подвохов в будущем. Проще говоря, сейчас это всё – *для личного спокойствия,* но может сильно пригодится в самом недалёком будущем.

</details>

Рекомендации:  
1. ТОЛЬКО стоковые прошивки ИЛИ crDroid / EvolutionX, где разработчиками предусмотрен спуфинг важных значений из build.prop, считываемых приложениями на предмет уровня безопасности сборки, непосредственно в самой прошивке "из коробки".  
Для хреновых прошивок, где этого не предусмотрено, существует модуль [Sensitive Props,](https://github.com/bugreportion/sensitive-props-module) по возможности подменяющий важные значения в некоторых строках (при том не модифицируя read-only строки, дабы избежать обнаружения Property Modified). Тем не менее, некоторые lineage-specific пропы могут быть по-прежнему доступными для чтения всем приложениям (это можно проверить в любой shell-оболочке без root-доступа, выполнив команду `getprop | grep 'lineage'` / или `'crdroid'`). Эта проблема решена пока что лишь в EvolutionX; этих пропов в принципе не существует на стоковых прошивках вендоров, поэтому они у нас в приоритете. Вы так же можете попробовать удалить их вручную, если это позволяет сделать файловая система на вашем девайсе.

### Инструкция
0. Имеем "чистую" от модулей систему с настроенным Kitsune Mask. Понадобятся приложения [Native Detector,](https://t.me/reveny1/45) [Native Test,](https://t.me/LSPosed/256) [Momo](https://github.com/apkunpacker/MagiskDetection/blob/main/Momo-v4.4.1.apk) и [Memory Detector.](https://github.com/rushiranpise/detection/blob/main/MemoryDetector_2.1.0.apk) От всех трёх четырёх скрываем root-права в Magisk Hide.
Устанавливаем ранее указанный модуль [Magisk OverlayFS.](https://github.com/HuskyDG/magic_overlayfs) Внимание: строго рекомендуется следовать [указанной выше инструкции.](https://github.com/begoniacommunity/list/blob/main/list/android.md#kitsune-mask) OverlayFS на официальном Magisk той же версии не работает. Перезагружаем устройство.
1. После установки модуля, в Терминале (Shell) выполните команду: `su -mm -c magic_remount_rw`.
2. **Если вы на кастомной прошивке,** переходим в файловый менеджер с поддержкой root (рекомендуется [Material Files](https://github.com/zhanghai/MaterialFiles)), переходим в папку /system/addon.d и удаляем файл(-ы) из этой папки. Меняем права для папки на 0000 (снимаем все галочки для всех юзеров). Визуально, в файловом менеджере, изменения в разрешениях могут не примениться, но на деле это работает.  
   - Если у вас файловая система ext4: перейдите к файлу build.prop в режиме текстового редактора и удалите все строки, содержащие `lineage`, `aosp` или `crdroid`. Сохраните изменения. Если изменения сохранены успешно, установите разрешения 0600 для файла build.prop. Если вы получили ошибку `I/O error`, редактирование build.prop для вас недоступно.
   - Дополнительно, если в этом есть необходимость, измените в файловой системе то, что хотели.
3. Как всё сделали, выполняем в Терминале: `su -mm -c magic_remount_ro`.
4. Проходим по пути /data/adb/modules/magisk_overlayfs и открываем текстовым редактором файл *mode.sh*, находим строчку `export OVERLAY_MODE=`, меняем значение с `0` на `2`.
5. Перезагружаемся и после проверяем, применились ли изменения. Если нет – повторяете предыдущий пункт, такое бывает.

Переходим в Native Detector – среди вывода обнаружений должно быть только приложение Native Test (да, такой вот тупизм). В приложении Native Test вывод должен быть Normal.  
Переходим в Momo – там вывод должен ограничиться максимум двумя обнаружениями: "Device running a custom ROM" и/или "Bootloader unlocked". Первое – результат невозможности скрытия lineage/aosp-specific строк, второе в объяснении не нуждается (только если вы не заблокировали загрузчик [с использованием avbroot,](#блокируем-загрузчик--скрываем-разблокированный) тогда это обнаружение **ложное**). Если будет что-то про режим дебаггинга, это отладка по USB, соответственно игнорируем.  
В Memory Detector должен быть вывод *Looks fine! Nothing is found.*

### Патчинг модулей
>
> [!IMPORTANT]
> Этот метод может не работать с хардкоднутыми модулями вроде Systemless BiTGApps. Для модулей, ничего не добавляющих/меняющих в системе и системных файлах соответственно (например, Play Integrity Fix или Zygisk Detach), эта процедура не требуется.

[Список модулей для Magisk можно найти здесь.](https://github.com/begoniacommunity/list/blob/main/list/android.md#%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D0%B8-%D0%B4%D0%BB%D1%8F-magisk)

1. Открываем архив с модулем и проходим по пути /META-INF/com/google/android, открываем текстовым редактором файл update-binary.
2. Перед завершением установки модуля (это последняя строчка, обычно `exit`) добавляем следующий скрипт:
```
OVERLAY_IMAGE_EXTRA=0     # number of kb need to be added to overlay.img
OVERLAY_IMAGE_SHRINK=true # shrink overlay.img or not?

# Only use OverlayFS if Magisk_OverlayFS is installed
if [ -f "/data/adb/modules/magisk_overlayfs/util_functions.sh" ] && \
    /data/adb/modules/magisk_overlayfs/overlayfs_system --test; then
  ui_print "- Add support for overlayfs"
  . /data/adb/modules/magisk_overlayfs/util_functions.sh
  support_overlayfs && rm -rf "$MODPATH"/system
fi
```
Если возникли проблемы с добавлением скрипта (например, строка завершения установки в update-binary вовсе отсутствует), добавьте его в customize.sh или install.sh.

3. Сохраняем отредактированный файл и переносим его в установочный архив с заменой. 
4. Устанавливаем модуль. Если всё успешно (вы правильно вставили скрипт, а модуль поддерживает установку через OverlayFS), в логе установки будет схожий вывод, как при установке самого модуля OverlayFS. 
5. Перезагружаем устройство и проверяем:
   - сам модуль, всё ли работает корректно;
   - обнаружение файлов в Memory Detector – должен быть вывод *Looks fine! Nothing is found.*
   
## [LSPosed Mod](https://github.com/mywalkb/LSPosed_mod/releases)
<sup>*(ссылка на скачивание в заголовке)*</sup>

С недавних пор разработка оригинального LSPosed была заброшена его автором. Спустя несколько месяцев нашелся энтузиаст, взявший на себя дальнейшее развитие уже своего форка. И он, внезапно, решил самую главную проблему LSPosed в контексте нашего занятия – его больше не обнаруживают приложения, к которым не применяется инъекция. Теперь можно спокойно ставить. Но помните, что некоторые отдельные приложения могут ругаться на инъекцию zygote, если применить к ним тот или иной LSPosed-модуль. После установки в настройках отключите пункт "Enable watchdog logs" и включите "Выключить подробные логи".

[Список модулей для LSPosed можно найти здесь.](https://github.com/begoniacommunity/list/blob/main/list/android.md#%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D0%B8-%D0%B4%D0%BB%D1%8F-lsposed)

### [Hide My Applist для LSPosed](https://github.com/Dr-TSNG/Hide-My-Applist/releases)
<sup>*(ссылка на скачивание в заголовке)*</sup>

1. Скачиваем и устанавливаем Hide My Applist, перезагружаем устройство.  
2. Проходим в LSPosed → Модули → активируем Hide My Applist (автоматически поставится галочка на Системный фреймворк). Перезагружаем устройство.  
3. Заходим в Hide My Applist, убеждаемся, что модуль активирован и системная служба работает. Переходим в Управление шаблонами → Создать шаблон с чёрным списком → выбираем всё, что нужно скрыть. Добавляем название для шаблона и сохраняем его.  
4. Возвращаемся в главное меню и открываем Управление приложениями. Там находите приложения, от которых нужно что-то скрыть. В меню каждого из них включаете скрытие и применяете созданный ранее шаблон (первый пункт в разделе "Конфигурация шаблонов").  

## Аккуратно инжектим ReVanced в оригинальный YouTube
> [!WARNING]
> Откровенно говоря – пока что очень спорное решение с root-версией Ютуба.  
> Да, действительно, следы монтирования модуля исчезнут, но пользоваться такой связкой очень проблематично: во-первых, холодный старт приложения становится в 2-3 раза дольше, чем обычно; во-вторых, при переходе в приложение YouTube через ссылку, запускается обычное, немодифицированное приложение (то есть без инжекта). Решать вам.  
> Если хотите идеальное и безболезненное скрытие, всегда есть альтернатива в виде [non-root версии ReVanced Extended.](https://github.com/MatadorProBr/revanced-extended-magisk-module/releases) 

1. Скачиваем и устанавливаем модуль [Zygisk Proc Injection,](https://github.com/HuskyDG/magic_proc_monitor) перезагружаем устройство.
2. Теперь скачиваем и устанавливаем [версию оригинального YouTube,](https://www.apkmirror.com/apk/google-inc/youtube/) которая уже используется в качестве базы для [сборки ReVanced Extended.](https://github.com/MatadorProBr/revanced-extended-magisk-module/releases) Скачиваем соответствующую root-версию.
3. Скачиваем модуль [YT Revanced Inject:](https://t.me/pixelifysupport/124919)
   1) Открываем архив с модулем и редактируем файл `module.prop`: меняем `version=` с `18.45.43` на версию выбранного вами клиента YouTube (например, `19.07.43`).
   2) Открываем ранее скачанный архив root-версии ReVanced Extended и достаём из него base.apk → переименовываем в revanced.apk и переносим с заменой в архив с модулем YT Revanced Inject.
4) Сохраняем все изменения и устанавливаем модуль.

## Блокируем загрузчик / скрываем разблокированный
На устройствах Google Pixel и OnePlus можно заблокировать загрузчик с установленной неофициальной прошивкой (и даже с root-правами). Это возможно и на других устройствах, поддерживающих кастомную подпись AVB (avb_custom_key) – с помощью программы [avbroot.](https://github.com/chenxiaolong/avbroot/blob/master/README.ru.md) Такая блокировка не поможет пройти Strong-сертификацию в Play Integrity, но для редчайшего набора приложений, проверяющих статус разблокировки, этого будет достаточно.  
Для других устройств есть менее надёжный, частично рабочий метод с модулем [BootloaderSpoofer](https://github.com/chiteroman/BootloaderSpoofer) для LSPosed. Активируйте его для приложений, обнаруживающих разблокированный загрузчик. Есть вероятность, что это поможет.