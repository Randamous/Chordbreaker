# Последние изменения: боёвка, настройки, UX

Обновлено: 21 июня 2026.

Этот файл фиксирует изменения, добавленные за последние несколько дней, чтобы не искать их по диалогу или git diff.

---

## Пакет правок 20 июня 2026

### Стаки пассивных предметов
- PASSIVE-предметы теперь могут повторно выпадать до `max_stack` и усиливают уже взятый экземпляр вместо занятия нового пассивного слота.
- `ItemSlotsPanel` показывает количество пассивки подписью `xN`, а tooltip добавляет `Кол-во: N/max` и строку `Текущий эффект: ...` с итоговыми числами.
- Checkpoint сохраняет пассивки в структурированном формате `{path, stack}`; старые сохранения со списком строк по-прежнему читаются как stack 1.
- Текущие пассивки масштабируются до 5 стаков: ауры/яд/шипы увеличивают урон, шансовые пассивки увеличивают шанс до 100%, `Семя Сердца` увеличивает бонус к выпадению сердечек.
- Проверка после правки: `item_database` 14/14, `reward_offer_queue` 9/9, `item_pickup_skill_upgrade_description` 7/7, полный `test_run` 139/139.

### Tooltip апгрейдов в слотах
- ItemSlotsPanel теперь пересобирает tooltip предмета через общий helper при добавлении и повышении уровня, а UPGRADE-слоты показывают текущий итог эффекта (Текущий эффект: ...) вместо устаревшего базового описания или preview следующего уровня.
- Карточки выбора предметов продолжают показывать current -> next; tooltip уже взятого апгрейда показывает именно текущий уровень и текущий итог.
- Добавлен тест item_pickup_skill_upgrade_description::test_upgrade_slot_tooltip_describes_current_level_result.

### Проверка скейлинга остальных навыков
- `AbilityResource` получил общий helper `get_scaled_aoe_radius(base_radius, owner_node)`, который читает `player` meta `aoe_radius_multiplier` только для навыков с тегом `АоЕ`.
- AoE radius-scaling подключён к `ArcaneBlastAbility`, `AuraAbility`, `BindingArrowAbility`, `ChargeDashAbility`, `FireballAutoAttack`/`fireball.gd`, `PoisonArrowAbility`, `SnareTrapAbility`, `WarCryAbility` и уже исправленному `ArrowRainAbility`.
- `SlashAutoAttack` получил теги `Ближний бой` и `Крит`, поэтому `Боевой Импульс` и `Критический Удар` теперь видят этот авто-навык через общий tag-контракт.
- Добавлен suite `ability_item_scaling`: проверяет общий AoE radius multiplier contract, mutable fireball explosion radius и скейлинг `SlashAutoAttack` от `Боевого Импульса`.

### Скейлинг Дождя стрел от предметов
- `ArrowRainAbility` теперь при касте передаёт зоне effective radius с учётом `player` meta `aoe_radius_multiplier` и effective damage из текущего `ability.damage`.
- Это подключает `Широкий Резонанс` к радиусу `Дождя стрел`; `Боевой Импульс` продолжает скалировать урон через общий путь `AbilityResource.damage`.
- Добавлен suite `arrow_rain_ability`: проверяет теги, базовый overlay radius и item-scaled radius/damage.

### Passive-ауры и Огненный Щит
- `PassiveComponent` больше не применяет `timed_aura` эффекты, пока у игрока активен `is_room_entry_invincible()` после перехода в комнату; таймеры не накапливают мгновенный урон после снятия щита.
- `Огненный Щит` переведён с `on_hurt` на постоянную timed-ауру: 8 огненного урона раз в секунду в радиусе 110px.
- Добавлен suite `passive_component`: проверяет suspend timed-аур во время room-entry shield и регистрацию `fire_shield` как timed-ауры.

### Boss encounter table
- Добавлен `BossEncounterTable`: boss-room выбирает encounter pool по `biome_id`, `dungeon_level` и `difficulty_id`, с fallback на `default` biome/difficulty.
- `BaseRoom._get_boss_scene()` теперь спрашивает `default_boss_encounter_table.gd`; прямой `MinotaurScene` остался только последним fallback.
- Добавлен второй boss encounter: `res://scenes/enemies/orc_warlord_boss.tscn`, наследует boss-FSM Минотавра и использует boss-маппинг кадров из `orc_warrior_frames.tres`.
- Archer floor band 5 на `hard`/`nightmare` теперь выбирает Орка-воеводу; `normal` получает смешанный pool Минотавр/Орк-воевода.
- Проверка после правки: `boss_encounter_table` 4/4, `base_room_boundaries` 7/7, полный `test_run` 130/130.

### Spawn-pool floor bands
- `RoomSpawnPoolTable` получил `floor_band_pools`: таблица выбирает самый высокий band, чей минимальный этаж не выше текущего `dungeon_level`.
- `legacy_spawn_pool_table.gd` и `archer_spawn_pool_table.gd` теперь имеют отдельные составы для этажей 1, 2, 3–4 и 5+; `level_2_pools` оставлен как fallback/compatibility поле.
- Проверка после правки: `room_spawn_pool_table` 5/5, `base_room_boundaries` 6/6.
- 21 июня 2026: START/SHOP/TREASURE больше не откатываются к NORMAL-пулу внутри `floor_band_pools`, если для них нет явного пула на этаже. Это чинит спавн врагов в стартовой комнате после перехода на следующий этаж через boss-portal. Проверка: `room_spawn_pool_table` 6/6, `room_rules` 4/4, `room_manager_routes` 11/11.

### Пересборка PASSIVE/UPGRADE rewards
- `ItemResource` получил поля `item_tags`, `upgrade_tags`, `is_legacy`, чтобы коллекция и будущие фильтры могли работать не только по типу предмета.
- Активный PASSIVE-пул расширен до 8 предметов: ледяная аура, огненный щит, шипы, сердечки, яд, рикошет, молния, вампиризм. Новые пассивки подключены через `PassiveComponent`, player meta и `BaseEnemy._heart_drop_chance()`.
- Активный UPGRADE-пул расширен до 10 предметов: мультивыстрел, урон, крит, отталкивание, скорость движения, уворот, блок, откаты, размер AoE, скорость атаки. Новые апгрейды дают preview через `get_upgrade_change_lines(...)`.
- Корневые legacy item definitions остаются в `ItemDatabase.get_all_items()` для коллекции/старых сейвов, но помечаются `is_legacy=true` и исключаются из `get_available_items()`.
- `player.gd` читает `block_chance`/`block_damage_reduction`, `arrow.gd` читает `aoe_radius_multiplier` и `knockback_strength_multiplier`.
- Проверка после правки: `item_database` 12/12.

### Выбор биома и сложности в городе
- `city_screen.tscn` получил строку `RunSetup` с `BiomeOption` и `DifficultyOption`; старт забега вызывает `GameManager.configure_run(...)` перед переходом к выбору персонажа.
- `GameManager` хранит выбранные `biome_id` (`legacy`, `archer`) и `difficulty_id` (`normal`, `hard`, `nightmare`), нормализует значения и восстанавливает их при загрузке checkpoint.
- `world.gd` пишет `biome_id`/`difficulty_id` в save data и `RoomData.meta`; выбор биома больше не привязан к персонажу.
- `BaseRoom` применяет сложность к `EnemyData.max_health`, `damage`, `speed`, а `SaveManager.record_run_result(...)` учитывает множитель meta-награды.
- Проверка после правки: `save_manager` 12/12, `base_room_boundaries` 3/3, полный `test_run` 115/115.

### Город, коллекция и meta-upgrades
- `res://scenes/ui/city/city_screen.tscn` теперь содержит реальные ноды layout, TabContainer, список улучшений и grid коллекции. Скрипт `city_screen.gd` больше не собирает основной экран целиком кодом, а только заполняет строки/карточки.
- Вкладки города явно подписываются через `TabContainer.set_tab_title(...)`; коллекция получила базовый фильтр по типу предмета и адаптивное число колонок.
- Добавлена ресурсная база meta-upgrades: `MetaUpgradeResource`, `MetaUpgradeDatabase` и `.tres` файлы в `res://data/meta_upgrades/`. Новые улучшения можно править в инспекторе, не выискивая параметры в `city_screen.gd`.
- Meta-upgrade `attack_speed` из города убран как слишком сильно влияющий на бой. Вместо него добавлены `skill_slots` (активные skill-слоты 2 -> 5) и `upgrade_slots` (уникальные UPGRADE-предметы 2 -> 4).
- `world.gd` синхронизирует лимиты через `_sync_inventory_slot_limits()`: `ItemDatabase` фильтрует офферы по лимитам, `AbilityComponent.max_ability_slots` ограничивает книги навыков, `ItemSlotsPanel` скрывает закрытые upgrade-слоты.
- Проверка после правки: `ability_component` 6/6, `item_database` 9/9, `save_manager` 12/12, полный `test_run` 114/114.
- 21 июня 2026: формула `SaveManager.calculate_meta_currency_reward(...)` стала компактнее: снижены вклад score/золота/уровня и бонус победы, difficulty multiplier сохранён. Пример `victory floor=5 score=1200 gold=200 level=8` теперь даёт 69 вместо прежних 138. В городе мета-валюта показывает `spark_icon.svg` через `MetaCurrencyIcon`, чтобы отличаться от золота забега. Проверка: `save_manager` 13/13.
- 21 июня 2026: закрыт дублирующий TODO про общий профиль сохранения. Account-level `meta_progress.sav` остаётся отдельным от run-слотов; добавлен тест, что удаление/переиспользование `save_slot_N.sav` не сбрасывает валюту, known-items и уровни meta-upgrades. Проверка: `save_manager` 14/14.

### Элитные враги
- 21 июня 2026: элитки получили единый visual/hitbox scale x1.25 через `EliteModifierResource`; live collision shapes обновляются после применения модификатора. HP bar status-иконка и раскраска aura по конкретному модификатору убраны, визуальная читаемость теперь через общий размер и `EliteAura`. Проверка: `elite_modifier_resource` 2/2, `enemy_hp_bar_statuses` 2/2.
- 21 июня 2026: добавлен первый спец-модификатор элит `shielded`: с этажа 2 `EliteModifierCatalog` может выбрать `res://data/elite_modifiers/shielded.tres`, а `BaseEnemy` иногда режет входящий урон через `elite_block_chance` / `elite_block_damage_reduction`. Источники урона могут обойти блок через option `ignore_elite_block=true`. Проверка: `elite_modifier_resource` 4/4, `enemy_damage_alert` 2/2, `hitbox_geometry` 2/2.
- 21 июня 2026: добавлен спец-модификатор элит `burning`: с этажа 2 каталог может выбрать `res://data/elite_modifiers/burning.tres`; модификатор пишет `elite_damage_aura_*`, а `BaseEnemy` создаёт `EliteDamageAuraTicker` и периодически наносит игроку урон рядом через `player.take_damage(...)`. Проверка: `elite_modifier_resource` 5/5, `enemy_damage_alert` 2/2, `enemy_hp_bar_statuses` 2/2, `hitbox_geometry` 2/2.

## Пакет правок 16 июня 2026

### Навыки лучницы
- `ItemDatabase.SKILL_ITEM_SLOT_LIMIT` увеличен до 5: стартовый `arrow_fan` считается skill-слотом, а книжные навыки занимают свободные активные слоты [2..5]. Поэтому после стартового веера и трёх полученных навыков лучница снова может увидеть пятый новый SKILL в наградах.
- Базовый cooldown `ArrowFanAbility` увеличен до 10.0с; skill-upgrade preview продолжает считать улучшения от live/current cooldown.
- Добавлен приобретаемый SKILL «Стягивающая стрела [Лучница]»: `binding_arrow_ability.gd`, `item_skill_binding_arrow.gd`, `binding_arrow_projectile.tscn/.gd`, регистрация в `ItemDatabase`.
- Стягивающая стрела летит по курсору, при попадании наносит прямой урон цели, выбирает точку удара, стягивает ближайших врагов в радиусе 110px и затем вызывает штатное `stun(1.0)`.
- Через `$imagegen` добавлены bitmap sprite-sheet ассеты `res://assets/generated/binding_arrow_projectile_sheet.png` и `res://assets/generated/binding_arrow_vines_sheet.png`. `BindingArrowProjectile` больше не подкрашивает общий `arrow.png` и не рисует временные линии лиан через `_draw()`: projectile, tether и bind-burst используют отдельные PNG-листы.
- `test_item_database.gd` покрывает сценарий `стартовый веер + 3 навыка -> новый пятый SKILL доступен`, состав archer skill pool и загрузку нового projectile.

---

## Пакет правок 7 июня 2026

### Проверка Godot-ai и ItemDatabase
- Godot-ai MCP проверен напрямую: активная сессия `horde-roguelite`, editor ready, полный `test_run` проходит `63/63`.
- `ItemDatabase` теперь `@tool class_name ItemDatabase` с экземплярными методами. Runtime-код создаёт базу через `load("res://systems/items/item_database.gd").new()`, чтобы тот же контракт работал и в Godot-ai/@tool тестах.

### Экран загрузки комнат
- `RoomManager.load_room()` теперь показывает программный `LoadingTransitionOverlay`: черный fullscreen, логотип, подпись типа комнаты и проценты в правом нижнем углу.
- Переходы используют разные tween-настройки по типам комнат: обычный fade, boss back/pulse, shop/treasure быстрый quad fade, secret cubic fade. Загрузка пока синхронная; проценты показывают этапы текущего `load_room()`, а не фоновую подгрузку ресурсов.
- Добавлен smoke-тест `room_manager_routes`, который проверяет создание overlay и финальное значение `100%`.

### Описания изменений апгрейдов
- `ItemDatabase.get_upgrade_change_description()` больше не хардкодит формулы конкретных UPGRADE по имени файла.
- Upgrade item scripts могут реализовать `get_upgrade_change_lines(current_level, next_level) -> Array[String]`; эти строки добавляются к карточке награды после общего заголовка `Апгрейд: ур. X -> Y/max`.
- Текущие апгрейды `multishot`, `crit`, `dodge` и `attack_speed` уже используют этот контракт, поэтому новые апгрейды должны описывать собственную дельту рядом со своей логикой уровня.

### Legacy UI cleanup
- `shop_screen.gd` помечен как legacy: актуальный магазин работает через `item_pickup_screen.gd` в shop-mode и pickup-сцены магазина.
- `item_choice_screen.gd/.tscn` помечены как legacy: актуальный экран наград — `item_pickup_screen.gd` с очередью offer-ов, shop-mode, replacement-flow и описаниями апгрейдов.
- `shop_screen.tscn` в проекте не найден; существует только `res://scenes/ui/shop_screen.gd`.

### Иконка пассивки лучницы
- Через `$imagegen` создана отдельная иконка `res://assets/generated/icon_archer_passive_reflexes.png`.
- `ArcherPassive` теперь задаёт `icon` в `_init()`, поэтому пассивный слот [7] и tooltip не остаются без изображения.

### Экран характеристик персонажа
- Добавлен `res://scenes/ui/player_stats_screen.gd`: экран показывает текущие health/max_health, speed, armor, crit, dodge, attack speed multiplier, multishot, level, gold и dungeon floor.
- `world.gd` и `boss_test_room.gd` открывают экран через input action `open_character_stats` (по умолчанию `C`). Экран не ставит игру на паузу, не блокирует input вне своей панели и закрывается через `C`, `Esc` или кнопку «Закрыть».
- `SettingsManager` добавляет action в список переназначаемых биндов как «Характеристики».

---

## Пакет правок 1 июня 2026

### Апгрейды существующих навыков
- Повторный выбор уже взятого SKILL больше не создаёт дубль в инвентаре: `world.gd` и `boss_test_room.gd` повышают уровень существующего skill item и соответствующего `AbilityResource`.
- `AbilityResource` хранит `ability_level` и `max_ability_level` (по умолчанию 5). Базовое масштабирование уровня: +20% damage и -5% cooldown за каждый уровень после первого; исходные значения сохраняются в meta `base_damage` / `base_cooldown_time`.
- Карточка награды для уже взятого SKILL теперь показывает, что именно изменится при выборе: `Улучшение: ур. X -> Y/max`, дельту урона и дельту отката. Текст собирается через `ItemDatabase.get_skill_upgrade_description()` и используется `ItemPickupScreen`.
- `ItemDatabase` теперь показывает уже взятые навыки в наградах только если их уровень ниже максимума. Если 4 skill-слота заняты, новые навыки скрываются, но апгрейды существующих навыков остаются доступными.
- `ItemDatabase.pick_offer()` больше не берёт случайные первые элементы после shuffle: оффер выбирается weighted-random по `get_offer_weight(item)`, где учитываются тип предмета и редкость. Сейчас веса типов: SKILL=1.0, PASSIVE=1.25, UPGRADE=1.4; веса редкости: COMMON=1.0, UNCOMMON=0.75, RARE=0.35, LEGENDARY=0.15.
- `DashAbility` теперь имеет cooldown=8с и при старте рывка вызывает полностью заряженный выстрел `AutoAttackAbility` по ближайшему врагу в радиусе 220px. Бонусная стрела не проверяет и не сбрасывает cooldown автоатаки; для других персонажей эффект не срабатывает, если в слоте [0] не `AutoAttackAbility`.
- Стартовый навык лучницы «Веер стрел» зарегистрирован как `item_skill_arrow_fan.gd` в `inventory.skills` с Lv1, поэтому книга навыка улучшает стартовый слот [1] до Lv5 без создания дубликата.
- Текущий SKILL-пул лучницы ограничен книгами: `arrow_fan`, `arrow_rain`, `dash`, `piercing_arrow`, `poison_arrow`, `snare_trap`; `fireball`, `teleport` и `charge_dash` больше не выпадают лучнице.
- Сохранение инвентаря для `skills` перешло с массива строк на массив словарей `{path, level}`; загрузка поддерживает старый строковый формат.
- `SaveManager.normalize_ability_order()` приводит сохранённый `ability_order` к полному уникальному порядку 5 активных слотов. `world.gd` использует эту нормализацию при загрузке и автосейве, чтобы старые/битые сейвы не сдвигали `SkillsBar` в игровом режиме.
- У лучницы `body_collision_offset`, `hitbox_offset` и `target_center_offset` подняты к визуальному центру `(0,-30)`. Поэтому hitbox больше не съезжает вниз, а `RoomEntryShield` при входе в комнату центрируется вокруг персонажа; `projectile_spawn_offset` оставлен на `(0,-29.4)`.
- `Player` разделяет ручную/debug-неуязвимость (`set_invincible`) и временную неуязвимость навыков (`set_ability_invincible`). Dash, Teleport и ChargeDash больше не выключают debug-переключатель «Неуязвимость» в `boss_test_room`.
- `ItemSlotsPanel` показывает уровень навыков так же, как уровень апгрейдов.
- Текст поверх иконок навыков стал читаемее: `AbilitySlot` увеличивает шрифт и добавляет чёрную обводку для KeyHint/CooldownLabel, а `ItemSlotsPanel` делает то же для Lv-меток.
- `RoomManager.load_room()` очищает группу `room_transient`, а снаряды, длительные зоны, ловушки и короткие VFX навыков добавляются в неё при создании. Поэтому ArrowRainZone, SnareTrapZone, стрелы, fireball и похожие временные эффекты больше не остаются в `World` после перехода между комнатами.

### Геометрия ловушки и удара орка
- `BaseEnemy` получил `get_hitbox_center()`, чтобы способности могли проверять попадание по фактическому центру хитбокса, а не по позиции ног.
- `SnareTrapAbility` теперь использует центр хитбокса врага для trigger/effect radius и игнорирует мёртвых врагов.
- У `orc_warrior_enemy.gd` сужена фактическая геометрия удара: swipe arc 105° и reach 96px вместо прежнего широкого сектора.

### Агро после урона
- `BaseEnemy.take_damage()` теперь включает damage-alert на 6 секунд и запоминает игрока как цель.
- `chase_state` не отпускает далёкого игрока обратно в patrol, пока damage-alert активен. Поэтому враги начинают бежать к игроку после полученного урона даже вне обычного `aggro_range`.

### Разведение толпы врагов
- `BaseEnemy.get_crowd_chase_direction()` добавляет ближним врагам separation от соседей и распределение по боковым слотам вокруг игрока, когда несколько врагов находятся рядом с целью.
- Толпа теперь определяется по соседям рядом с самим врагом, а не только рядом с игроком. Поэтому плотная пачка начинает расходиться уже на подходе издалека, включая `boss_test_room`.
- `chase_state` для melee-врагов использует это направление вместо прямого движения в центр игрока. Враги всё ещё сближаются с целью, но крайние враги заметнее расходятся по диагоналям и меньше кучкуются в одной точке.
- Во время cooldown после атаки melee-враг не стоит в общей куче, а продолжает мягко перестраиваться в своём lane.
- В движущихся ветках `chase_state` теперь явно включается `walk`-анимация. Это чинит случай, когда враг после атаки оставался визуально в `idle`/`attack`, хотя уже снова догонял игрока.

### Таймер slow-иконки
- `SlowIndicator` теперь хранит `remaining` и обновляет его каждый кадр. `EnemyHPBar` считывает это поле и показывает убывающее заполнение slow-иконки, как у остальных временных статусов.
- `EnemyHPBar` теперь работает как `top_level` Control с высоким `z_index` и каждый кадр следует за родителем. HP/status bar врага больше не должен перекрываться спрайтом другого врага.

### Музыка комнат и boss-room
- Добавлены треки `res://assets/audio/dark_synth_forest_2.mp3` и `res://assets/audio/cartridge_grief.mp3`.
- `world.tscn` получил `AudioStreamPlayer` `MusicPlayer`; `world.gd` переключает музыку при входе в комнату. Обычные комнаты используют `MUSIC_BIOME_TRACKS`, boss-room — `MUSIC_BOSS_TRACKS`; словари поддерживают переопределения по `biome_id` и `dungeon_level`.
- Сейчас default-тема всех биомов — `dark_synth_forest_2.mp3`, boss-room — `cartridge_grief.mp3`, оба трека запускаются с `AudioStreamMP3.loop = true`. При pause menu музыка продолжает играть, но ducking снижает громкость с -10 dB до -18 dB. В аудио-настройках добавлена опция `mute_music_unfocused`: если включена, музыка ставится на паузу, когда окно игры неактивно, и возвращается при фокусе окна.
- Boss-room encounter для archer-biome больше не использует обычный `orc_warrior_enemy.tscn`; поздний archer-boss теперь отдельная сцена `orc_warlord_boss.tscn`, ранние/legacy boss-room остаются на `minotaur_enemy.tscn`.

### Музыка главного меню
- В `res://scenes/ui/main_menu/main_menu.tscn` добавлен `AudioStreamPlayer` `MenuMusic` с треком `res://assets/audio/gravel_glass_reverb.mp3`.
- `main_menu.gd` включает `AudioStreamMP3.loop = true` и запускает трек при входе в главное меню; отдельного music-bus пока нет, используется `Master`.

### Проникающая стрела лучницы и иконки навыков
- `res://systems/abilities/abilities/piercing_arrow_ability.gd`: добавлен приобретаемый навык лучницы «Проникающая стрела». После 1.0с подготовки с attack-анимацией лучница выпускает стрелу вперёд по курсору.
- Навык использует колонну `ray`: range=320px, width=92px, наносит 40 урона всем врагам в зоне. Теги: `Крит`, `Снаряд`, `АоЕ`; cooldown=16с.
- `res://systems/items/definitions/skills/item_skill_piercing_arrow.gd` и `ItemDatabase`: добавлена книга навыка «Проникающая стрела [Лучница]» только для `archer`, слот назначения — свободные активные слоты [2..5].
- Через `$imagegen` обновлён набор иконок навыков: 48x48 `res://assets/generated/icon_*.png` с общей рамкой персонажа и уникальным символом каждого навыка. Для SKILL-предметов добавлены 32x32 glass-sphere иконки `res://assets/generated/items/icon_item_skill_*.png`.
- Через `$imagegen` создан sprite-sheet `res://assets/generated/piercing_arrow_sheet.png`: летящая стрела с широким зелёным лучом.

### Tooltip вне active scene tree
- `res://scenes/ui/tooltip_trigger.gd`: `_get_tooltip()`, `_on_mouse_entered()` и `_on_mouse_exited()` теперь проверяют `is_inside_tree()` и валидность родительского Control перед обращением к `get_tree()`/absolute paths. Это убирает ошибку `get_first_node_in_group` на null и предупреждение Godot про absolute paths вне active scene tree.

### Замедляющая ловушка лучницы
- `res://systems/abilities/abilities/snare_trap_ability.gd`: добавлен приобретаемый навык лучницы «Замедляющая ловушка». Точка выбирается рядом с лучницей в пределах 170px; через 1.0с ловушка заряжается, затем живёт 10.0с и срабатывает один раз, когда враг входит в малый радиус 36px.
- При срабатывании ловушка наносит 10 урона всем врагам в радиусе 92px и один раз применяет slow 0.6 на 5.0с (-40% скорости) к тем, кто был в зоне в момент активации.
- `res://systems/enemy/base_enemy.gd`: добавлен общий `apply_slow(multiplier, duration)` и `get_move_speed(scale)`. `patrol_state`, `chase_state`, `charge_state` и boss movement states теперь учитывают этот multiplier.
- `res://systems/items/definitions/skills/item_skill_snare_trap.gd` и `ItemDatabase`: добавлена книга навыка «Замедляющая ловушка [Лучница]» только для `archer`, слот назначения — свободные активные слоты [2..5].
- `res://scenes/world/ability_range_overlay.gd`: добавлена форма `snare_trap`, которая показывает дальность постановки, малый радиус активации и больший радиус эффекта.
- Через `$imagegen` создана иконка навыка `res://assets/generated/icon_snare_trap.png`; сама ловушка и взрыв используют кодовый VFX между капканом, руной и опутывающими лозами.

### Обновление boss_test_room
- `res://scenes/world/boss_test_room.gd`: левая debug-панель переведена на `ScrollContainer`, чтобы помещались все враги, предметы и тестовые действия.
- Список кнопок предметов теперь строится из `ItemDatabase.get_all_items()`, поэтому новые навыки/предметы появляются в тестовой комнате автоматически. Кнопки предметов показывают иконку и tooltip с кратким описанием.
- Добавлена кнопка «Сбросить кулдауны», которая обнуляет `cooldown_remaining`, выставляет `is_ready=true` и отправляет `cooldown_changed` для всех навыков игрока.
- Добавлены debug-кнопки «Неуязвимость» и «Повысить уровень»: первая переключает `player.set_invincible()` и сохраняет состояние при респавне в тестовой сцене, вторая добавляет XP до следующего level-up и открывает обычный экран награды.
- `SkillsBar` в основной сцене и `boss_test_room` выровнен по правому краю своей области, чтобы при расширении панель навыков уходила влево.
- Из pause menu убрана кнопка «Выбор персонажа». Пауза теперь оставляет быстрые действия: продолжить, настройки, выход в главное меню и выход из игры.

### Очередь экранов выбора предметов
- `world.gd` и `boss_test_room.gd` теперь защищены от вложенного показа `ItemPickupScreen`: если предмет во время выбора сам вызывает новый level-up/offer (пример: Чёрная Жемчужина даёт XP), следующий экран попадает в `_queued_item_offers` и показывается только после завершения текущего выбора.
- `item_pickup_screen.gd` в shop-mode больше не продолжает обновлять магазинные контролы, если купленный предмет переключил тот же экран в обычный выбор/замену.
- Toast «предмет получен» больше не блокирует клики по экрану выбора: его CanvasLayer работает на паузе, а все Control-узлы внутри используют `MOUSE_FILTER_IGNORE`.
- В `enemy_state.gd` добавлен общий `_move_speed(scale)`, а enemy/boss movement states используют его вместо локальных тернарных выражений скорости. Это сохраняет поддержку slow и убирает parse-ошибки типизации в Godot 4.

---

## Пакет правок 31 мая 2026

### Веер стрел
- `res://systems/abilities/abilities/arrow_fan_ability.gd`: базовый cooldown увеличен с 5с до 7с, а `get_range_info()` и реальные стрелы используют общий `arrow_range = 220px`.
- `res://scenes/skills/arrow.gd`: добавлен `max_distance` и счётчик пройденного пути, поэтому стрелы веера удаляются по дальности, а не только по lifetime.
- `res://scenes/skills/arrow.gd`: стрелы `ArrowFanAbility` больше не проходят сквозь врагов; после первого попадания отключают `monitoring`, наносят impact AoE в радиусе 56px, включают стандартизированный forced knockback через `force_hurt_knockback=true` + `knockback_strength=0.75` и удаляются.
- `res://systems/enemy/base_enemy.gd`: forced knockback теперь имеет общий контракт: caller передаёт направление и `options.knockback_strength`, а `BaseEnemy.resolve_knockback()` нормализует направление и умножает силу на `EnemyData.knockback_force`. Это же правило использует заряженная `auto_arrow`; её impact splash увеличен с 48px до 72px.
- `res://systems/abilities/abilities/auto_attack_ability.gd`, `res://scenes/player/player.gd`, `res://scenes/audio/audio_manager.gd`: при достижении 100% заряда автострелы проигрывается отдельный звуковой сигнал, а спрайт игрока мерцает до отпускания выстрела.

### Дождь стрел
- `res://systems/abilities/abilities/arrow_rain_ability.gd`: добавлен новый приобретаемый навык лучницы. На 1.0с кастует выбранную область, блокирует движение игрока и проигрывает attack-анимацию, затем 4.0с наносит 10 урона раз в секунду всем врагам в радиусе 110px.
- `res://systems/items/definitions/skills/item_skill_arrow_rain.gd` и `ItemDatabase`: добавлена книга навыка «Дождь стрел [Лучница]» только для `archer`, слот назначения — свободные активные слоты [2..5].
- Через `$imagegen` создан и подключён sprite-sheet падающих стрел `res://assets/generated/arrow_rain_sheet.png`; зона способности показывает предупреждающий круг до начала урона и анимацию стрел во время действия.
- `res://scenes/player/player.gd`: добавлен `lock_movement_for_seconds(duration)`, чтобы способности с подготовкой могли временно запрещать ручное движение без отдельной input-lock логики.

### Новый Shift-навык лучницы
- `res://systems/characters/characters/archer_character.gd`: survival-слот [6] лучницы теперь использует `ArcherSprintAbility`, а не `DashAbility`.
- `res://systems/abilities/abilities/archer_sprint_ability.gd`: «Легкий шаг» на 1.5 секунды повышает текущую speed на 100% и добавляет collision exceptions с врагами из группы `enemies`, чтобы игрок мог проходить сквозь них; cooldown=12с.
- При старте и окончании эффекта вызывается `ArcherPassive.apply_passive(player)`: при +100% speed шанс уворота становится 60%, затем возвращается к базовому значению.
- `player.take_damage()` при успешном dodge показывает индикатор «Уклонение».
- Для «Легкого шага» добавлен визуальный след: призрачные afterimage-копии текущего кадра персонажа и небольшие fading-партиклы позади движения.
- Добавлен suite `res://tests/test_archer_sprint_ability.gd`; полный Godot-ai `test_run` проходит 22 теста / 0 failures.

---

## Пакет правок 28 мая 2026

### Rolling log действий тестера
- Добавлен autoload `TesterActionLog` (`res://systems/debug/tester_action_log.gd`). Он пишет последние 400 событий в `user://tester_action_log.txt`, сбрасывает файл каждые 0.5 секунды и дополнительно после важных событий.
- В лог попадают: старт сессии, смена сцены, heartbeat раз в секунду, позиция/HP игрока, текущая комната, пауза, переходы через двери, teleport с карты, зачистка комнаты, level-up, выбор/замена предметов, victory/game over и основные input actions.
- Физический путь файла в Windows выводится в первой строке лога через `ProjectSettings.globalize_path()`. Тестер может прислать именно `tester_action_log.txt` после зависания; новые строки находятся внизу.
- `RoomManager` и `world.gd` обращаются к логгеру через `/root/TesterActionLog` с guard-ами, чтобы изолированные smoke-тесты могли инстанцировать классы без autoload.

### Плавное следование камеры
- `SettingsManager.DEFAULTS` получил `camera_smoothing_enabled = true` и `camera_smoothing_speed = 4.5`; `SettingsManager.apply_camera_settings(camera)` централизованно применяет параметры к `Camera2D`.
- `Player` применяет настройки к дочерней `Camera2D` в `_ready()` и обновляет их по `SettingsManager.settings_changed`, поэтому `world.tscn` и `boss_test_room.tscn` реагируют на изменения настроек без отдельной логики.
- В `settings_screen.gd` добавлена вкладка `Камера`: чекбокс плавного следования и слайдер скорости догоняния. Меньшее значение даёт более заметное запаздывание.
- Дефолт камер в `world.tscn`, `boss_test_room.tscn` и тестовых `forest_*` сценах выставлен на `position_smoothing_speed = 4.5`; `shop_test.gd` применяет общий профиль к программно созданной камере.

---
## Пакет правок 27 мая 2026

### Надёжность подбора предметов и XP-орбов
- `res://scenes/rooms/room_item_pickup.gd` теперь включает `body_entered` deferred после physics-кадров, а при валидном касании сразу отключает `monitoring` и вызывает `_collect()` deferred. Это убирает риск выдачи предмета/паузы UI прямо внутри physics-сигнала.
- `res://scenes/rooms/shop_item_pickup.gd` защищён от повторного `body_entered` при покупке: после списания золота pickup отключает `monitoring`, отсоединяет сигнал и завершает сбор deferred.
- `res://systems/enemy/base_enemy.gd` спавнит XP-орбы дочерними узлами текущей комнаты, а не в `current_scene`; `RoomManager.load_room()` дополнительно чистит группу `xp_orb` при смене комнаты.
- `res://scenes/xp_orb/xp_orb.gd` стал одноразовым: после первого контакта с игроком отключает monitoring/signal, чтобы XP не начислялся повторно.
- При загрузке сохранения `world.gd` теперь выставляет `XPBar.value = xp`, а не сбрасывает шкалу в `0`; это убирает ложное ощущение, что после загрузки опыт внезапно прилетел первым же orb-ом.

### Уменьшение лучницы и archer-biome врагов
- Лучница получила `CharacterResource.visual_scale = Vector2(0.7, 0.7)`; её body/hitbox, `target_center_offset` и `projectile_spawn_offset` уменьшены примерно на 30%.
- `GameManager.apply_character_to_player()` теперь применяет `visual_scale` через `Player.apply_character_visual_scale()`.
- Гоблин-лучник, волк, светлячок-дух и орк-воин уменьшены примерно до 70% прежнего визуального размера; вместе с ними уменьшены hitbox/body, AttackArea (`data.attack_range`), hp-bar offsets/sizes и специальные точки/радиусы: arrow origin у гоблина-лучника, explosion radius у светлячка, melee reach у орка-воина.
- Aggro/detection range оставлен прежним, чтобы уменьшение модели не делало врагов заметно более слепыми.
### Спрайты переходов вместо дверей
- `Door` остаётся `Area2D`-триггером перехода, но визуально теперь показывает зелёный pixel-art frame atlas `res://assets/generated/door_transition_arrows_anim.png`: широкие портальные стрелки появляются, движутся к выходу и растворяются уже внутри кадров, без программного сдвига Sprite2D.
- `BaseRoom` больше не рисует красные/зелёные дверные прямоугольники в стенах: закрытый или отсутствующий проход рисуется как сплошная стена, открытый проход — как разрыв стены плюс transition sprites.
- `Door.close()` скрывает все sprites и отключает collision; `Door.open()` включает collision и показывает только transition sprites. Legacy `SpriteOpen/SpriteClosed` не отображаются.
## Пакет правок 26 мая 2026

### Pixel HP bar и иконки статусов врагов
- `EnemyHPBar` теперь использует редактируемую тему `res://scenes/enemies/ui/enemy_hp_bar_theme.tres` и sprite atlas `res://assets/ui/enemy_hp_status_atlas.png`, сгенерированный через `$imagegen`: каменная рамка, тёмная подложка и красное заполнение HP.
- HP bar собирается из отдельных regions левого края, тайлящейся середины и правого края, поэтому ширина динамическая и редактируемая. Под полосой HP добавлена полка квадратных статусных иконок с таймером оставшейся длительности и эффектом заполнения снизу вверх.
- Новый UI API статусов: `show_status(id, duration)`, `refresh_status(id, duration)`, `clear_status(id)`, `set_statuses([...])`; поддерживаемые id: `poison`, `stun`, `slow`, `haste`, `buff`, `debuff`.
- Старая gameplay-логика эффектов не переписана: текущие `PoisonDoT` и `StunIndicator` только считываются HPBar для отображения таймера.

### Физика мертвых врагов
- `BaseEnemy._die()` теперь сразу отключает физическое тело, `Hitbox`, `DetectionArea` и `AttackArea`, до ожидания death-анимации.
- `dead_state.gd` повторно вызывает общий helper, поэтому враг во время анимации смерти остается только визуальным объектом и больше не блокирует снаряды.
- Снаряды игрока должны взаимодействовать только с живыми `enemy_hitbox`; мертвые враги остаются в группе до `queue_free`, но их `Area2D` уже не monitorable/monitoring и с нулевыми collision layer/mask.

---

## Пакет правок 25 мая 2026

### Дроп и отображение лута на карте
- Шанс дропа `heart_pickup` всё ещё зависит от HP игрока, но дополнительно уменьшается от общего количества лежащих сердечек во всех комнатах (`RoomData` meta `heart_pickups`).
- На полноэкранной карте (`M`) значки особенностей рисуются растровыми pixel-art PNG только для посещённых комнат: сердечко для `heart_pickups`, кожаная мошна для обычного неподобранного лута, монетка для магазина.
- Магазинные товары сами по себе не включают значок лута на карте; текущая комната дополнительно отмечается `!` в центре сетки слотов.

### Надежность экрана выбора предмета
- `res://scenes/ui/item_pickup/item_pickup_screen.gd` теперь полностью сбрасывает состояние выбора при каждом `show_offer()` / `show_replace_offer()` и убирает старые shop-controls из обычного экрана награды.
- Выбрать предмет можно кликом по всей карточке, не только по кнопке `Взять`; после выбора карточки блокируются от повторного emit до следующего открытия.
- Это закрывает случаи, когда popup `1 из 3` визуально открыт, но маленькая кнопка выбора не получает событие.

### Защита refresh-пикапа магазина
- `res://scenes/rooms/shop_refresh_pickup.gd` теперь отключает `body_entered` после первого валидного входа игрока и переармируется только после выхода игрока из safe radius (`REARM_CLEAR_RADIUS = 96`).
- Стояние на пикапе больше не запускает бесконечные обновления и не списывает всё золото; тот же refresh переармируется только после отхода и требует нового входа в зону.
- Обновление товаров вызывается deferred, чтобы не создавать/настраивать новые `Area2D` прямо внутри physics-сигнала.
- Collision layer/mask не менялись: refresh-пикап по-прежнему использует `CollisionLayers.PLAYER_TRIGGER_MASK`.

---

## Пакет правок 24 мая 2026

### Дроп сердечек из врагов
- `BaseEnemy._die()` теперь дополнительно пробует создать `heart_pickup` рядом с местом смерти врага.
- Базовый шанс зависит от текущего HP игрока: при полном здоровье 0.5%, при `<= 1` сердцу HP 15%, в остальных случаях 2%; итоговый шанс дополнительно уменьшается от количества уже лежащих сердечек во всех комнатах.
- Сердечко восстанавливает 1 сердце (`20` HP) и не подбирается/не исчезает, если здоровье уже полное.
- Состояние лежащих сердечек хранится в `RoomData` meta `heart_pickups`, поэтому они восстанавливаются при повторном входе в комнату.
- На полноэкранной карте (`M`) сердечки отображаются отдельным значком сердца, а не как общий лут.

### Хитбоксы и target boxes персонажей
- `CharacterResource` расширен полями `target_center_offset` и `projectile_spawn_offset`.
- `Player.apply_character_targeting()` применяет target center, `ProjectileSpawn/get_aim_origin()` и позицию `RoomEntryShield` из выбранного персонажа.
- Лучница: hitbox/body опущены вниз и укорочены до плеч/ног спрайта: 24x54 @ (0,-16), target=(0,-18), projectile=(0,-42).
- Маг: hitbox/body по 32px-спрайту: 20x31 @ (0,-30), target/projectile=(0,-30).
- Воин: hitbox/body по 32px-спрайту: 24x28 @ (0,-28), target/projectile=(0,-28).

### Клик по UI не запускает автоатаку
- `Player` помечает ЛКМ, начатую над UI, через `block_left_mouse_auto_attack_for_ui()` / `is_left_mouse_blocked_by_ui()`.
- `AbilityComponent` использует `_is_auto_attack_button_pressed()` вместо прямого `Input.is_mouse_button_pressed(MOUSE_BUTTON_LEFT)` и игнорирует ЛКМ, если курсор над `Control` или клик начался на UI.
- Это защищает клики по иконкам навыков и другому интерфейсу от параллельного выстрела/заряда автоатаки.

### Клик по иконкам навыков
- `AbilitySlot` теперь различает короткий ЛКМ-клик и drag: drag начинается после движения мыши на 8px.
- Клик по активному слоту с key action запускает `player.begin_aiming_ability_from_ui()` и включает locked overlay как в режиме «Прицелиться и подтвердить».
- Автоатака/пассивка без key action не активируются кликом; survival-слот с `skill_2` поддерживается.

### Anti-stick melee для врагов
- `BaseEnemy` получил helper-ы `distance_to_player_target()`, `is_player_body_contact()` и `is_player_in_melee_reach()`.
- Melee-враги в `chase_state` теперь считают физический контакт с телом игрока достаточной melee-дистанцией: они перестают давить в игрока и переходят в атакующую цепочку после cooldown.
- `charge_state` завершает melee-выпад в `attack` при контакте с игроком, а `attack_state` наносит fallback-урон при reach или контакте тел.
- Коллизии не отключались: игрок по-прежнему блокируется врагами, враги не должны залипать в бесконечном движении в тело игрока.

### Разные хитбоксы игровых персонажей
- `CharacterResource` получил поля `body_collision_size/body_collision_offset` и `hitbox_size/hitbox_offset`.
- `GameManager.apply_character_to_player()` применяет эти поля через `player.apply_character_collision()`.
- `Player` пересоздаёт `CapsuleShape2D` отдельно для физического тела и `Hitbox/CollisionShape2D`; размеры <= 0 игнорируются.
- Текущие значения: лучница 26x52 @ (0,-29), маг 24x50 @ (0,-29), воин 34x58 @ (0,-31).

### Телепорт с карты через маршрут и двери
- `res://scenes/world/world.gd`
  - `_on_map_teleport(target_room)` больше не вызывает `room_manager.load_room(target_room)` напрямую и не ставит игрока в центр комнаты.
  - Теперь делегирует переход в `room_manager.teleport_to_room_via_open_route(target_room)`.
  - Если открытого маршрута нет, телепорт отменяется и пишет warning.
- `res://systems/rooms/room_manager.gd`
  - Добавлен `teleport_to_room_via_open_route(target_room)`.
  - Метод ищет BFS-маршрут от текущей комнаты к выбранной только через реально проходимые двери.
  - Целевая комната загружается так, будто игрок дошёл до неё пешком и вошёл через последнюю дверь маршрута.
  - Игрок ставится через существующий `_place_player_at_entry(enter_direction)`, а не в центр.
  - Перед телепортом выставляется `_transition_cooldown = 1.0`, чтобы открытая дверь рядом со spawn-позицией не отправляла игрока обратно мгновенно.

### Правила проходимости для map teleport
- Проходимыми считаются двери из комнат, которые уже `cleared`, а также автооткрытые типы комнат: START, SHOP, TREASURE.
- Закрытые двери боевых неочищенных комнат не используются при поиске маршрута.
- Правило вынесено в `res://systems/rooms/room_rules.gd`: `BaseRoom._build_door_states()` и `RoomManager._is_room_door_open()` используют один источник истины.

### Направленные анимации и боевая цепочка врагов
- `res://systems/enemy/base_enemy.gd`
  - Общий стандарт directed-анимаций перенесён в базового врага.
  - Базовые анимации: `idle`, `walk`, `hurt`, `death`, `charge_windup`, `charge`, `attack`.
  - Опциональные северные варианты: `*_back` / `*_back_diag`.
  - Без `*_back` базовый спрайт считается юго-западным и зеркалится вправо; с `*_back` северные направления используют back-анимацию и зеркалятся влево.
  - Дублирующие `play_anim()` / `flip_toward()` убраны из `wolf_enemy`, `goblin_archer_enemy`, `firefly_spirit_enemy`, `orc_warrior_enemy`.
- FSM обычных и ranged-врагов стандартизирован:
  - `charge_state.gd`: melee теперь всегда проходит `charge_windup` → `charge` → `attack`.
  - `attack_state.gd`: фактический melee-удар происходит в `attack`.
  - `ranged_attack_state.gd`: дальники проходят `charge_windup` → `charge` → `attack`, выстрел происходит на фазе `attack`.

### Иконка проекта
- Через `$imagegen` создана новая квадратная иконка игры: портрет улыбающейся лучницы по референсу с экрана выбора персонажа.
- Файл: `res://assets/generated/icon_game_archer_smile.png`.
- `ProjectSettings.application/config/icon` переключён с `res://icon.svg` на новую PNG-иконку.

### Синхронизация документации с кодом
- Обновлены docs по персонажам: `sprite_frames_path`, направленные анимации игрока, корректный `skill_6` для 5-го активного слота.
- Обновлены docs по предметам: актуальный контракт `ItemResource`, `ItemDatabase.get_available_items()/pick_offer()`, текущие скорости XP-орбов.
- Обновлены docs по мета-системам: актуальные поля `GameManager`, сигнатуры `SaveManager`, порядок autoloads.
- Обновлены docs по врагам: добавлены `wolf`, `goblin_archer`, `firefly_spirit`; `orc_warrior` теперь подключён только во временный `archer` biome.


### Временное разделение биомов по персонажу
- `world._init_rooms()` проставляет всем комнатам `biome_id` по выбранному персонажу: лучница → `archer`, маг/воин → `legacy`.
- `BaseRoom` выбирает spawn pools по `biome_id`:
  - `archer`: только новые враги `goblin_archer`, `wolf`, `firefly_spirit`, `orc_warrior`.
  - `legacy`: только старые враги `goblin`, `orc`, `archer`, `bomber`, `summoner`.
- Boss-room pool тоже разделён по биомам исторически, но актуальный override в `base_room.gd` сейчас для всех биомов возвращает `minotaur_enemy`.
- Это временная привязка к персонажу; позже биом должен выбираться игроком в меню.

### Cleanup dead_state XP
- Удалена мёртвая `_spawn_xp()` из `res://systems/enemy/fsm/states/dead_state.gd`.
- XP/gold остаются централизованными в `BaseEnemy._die()` через `_spawn_xp_orbs()` и `_spawn_gold()`.

### CollisionLayers
- Добавлен единый файл `res://systems/collision_layers.gd`.
- `WORLD`, `PLAYER`, `ENEMY` и `PLAYER_TRIGGER_MASK` больше не копируются локальными `1 | (1 << 1)` в дверях, пикапах, XP/gold orb, enemy_arrow и enemy detection masks.
- Новые `Area2D`, которые ловят игрока через `body_entered`, должны использовать `CollisionLayers.PLAYER_TRIGGER_MASK`.

### Smoke-тесты RoomRules / CollisionLayers
- Добавлен первый Godot-ai test suite: `res://tests/test_room_rules.gd` (`suite_name() == "room_rules"`).
- Покрывает auto-open правила START/SHOP/TREASURE, closed-door behavior для uncleared NORMAL, открытие cleared-комнат, проверку наличия connection и базовые значения `CollisionLayers`.
- Проверка: `test_run(verbose=true)` проходит этот suite: 4 теста / 15 assertions / 0 failures.
### Smoke-тесты AbilityComponent
- Добавлен Godot-ai test suite: `res://tests/test_ability_component.gd` (`suite_name() == "ability_component"`).
- Покрывает контракт фиксированных 8 слотов: auto attack в [0], book abilities только в [2..5], overflow без расширения массива, bounds для `replace_ability()`.
- Полный `test_run(verbose=true)` теперь проходит 8 тестов / 38 assertions / 0 failures.
### Smoke-тесты RoomManager route/map teleport
- Добавлен Godot-ai test suite: `res://tests/test_room_manager_routes.gd` (`suite_name() == "room_manager_routes"`).
- Покрывает route-level поведение map teleport: маршрут через cleared rooms, отказ через uncleared NORMAL, auto-open SHOP/TREASURE, открытый обход вокруг закрытой комнаты и отказ для unvisited target.
- Полный `test_run(verbose=true)` теперь проходит 13 тестов / 46 assertions / 0 failures.
### Smoke-тесты SaveManager
- Добавлен Godot-ai test suite: `res://tests/test_save_manager.gd` (`suite_name() == "save_manager"`).
- Покрывает настоящий save/load roundtrip в `user://save_slot_2.sav`, delete/reset header, migration defaults, active slot bounds и `format_play_time()`.
- Suite делает backup/restore всех `user://save_slot_*.sav`, чтобы тест не оставлял следов в пользовательских сейвах.
- Полный `test_run(verbose=true)` теперь проходит 19 тестов / 82 assertions / 0 failures.
### Архитектурный обзор и ToDo
- По best practices Godot отмечены ближайшие технические задачи:
  - разгрузить `world.gd` на более маленькие runtime-контроллеры;
  - единый источник правил комнат/дверей уже введён как `RoomRules.gd`;
  - collision layers/masks уже централизованы в `res://systems/collision_layers.gd`;
  - перевести spawn pools из `base_room.gd` в данные/ресурсы;
  - первые smoke-тесты для `RoomRules`/`CollisionLayers`, `AbilityComponent`, route-level map teleport и `SaveManager` уже добавлены, дальше покрыть защиту от мгновенного возврата после teleport;
  - почистить мёртвый/legacy код или явно вынести его в архивную зону.

---

## Invulnerability игрока

### Урон по игроку
- `res://scenes/player/player.gd`
- После получения урона игрок получает 1 секунду неуязвимости к повторному урону.
- Во время этой неуязвимости игрок мигает, чтобы состояние было видно в бою.
- Логика разделена по типам invul, чтобы обычная неуязвимость от урона не конфликтовала с защитой при входе в комнату.

### Вход в новую комнату
- `res://scenes/world/world.gd` вызывает `player.grant_room_entry_invincibility()` после входа/загрузки комнаты.
- Игрок неуязвим первые 3 секунды после входа в комнату или до первого нажатия любой клавиши/мыши/геймпада.
- Для этой защиты рисуется сферический щит вокруг игрока.
- Враги не замечают игрока, пока активна именно room-entry invulnerability.

### Где враги проверяют защиту входа
- `res://systems/enemy/base_enemy.gd`: `get_player()` пропускает игрока с `is_room_entry_invincible()`.
- Прямые проверки босса/взрывника также учитывают это состояние:
  - `res://scenes/enemies/boss_dash_state.gd`
  - `res://scenes/enemies/boss_cone_state.gd`
  - `res://systems/enemy/fsm/states/explode_state.gd`
- Старые враги из `res://scenes/enemy/` удалены 2026-05-19; актуальные враги живут в `res://scenes/enemies/`.

---

## Stagger врагов

### Правило
- Обычный враг получает stagger, если отдельная атака наносит не меньше 30% от его максимального здоровья.
- Stagger длится 0.5 секунды.
- Повторное срабатывание возможно не чаще одного раза в 1.5 секунды.
- На боссов stagger не действует.

### Ограничения
Stagger не срабатывает, если враг уже выполняет важное действие:
- `attack`
- `charge`
- `rangedattack`
- `summon`
- `explode`
- `dead`

### Реализация
- `res://systems/enemy/base_enemy.gd`
  - `STAGGER_DAMAGE_FRACTION = 0.3`
  - `STAGGER_COOLDOWN_SECONDS = 1.5`
  - `_ensure_stagger_state()` добавляет состояние программно.
  - `_should_stagger(amount, state_before_hit)` решает, можно ли остановить врага.
- `res://systems/enemy/fsm/states/stagger_state.gd`
  - состояние паузы на 0.5 секунды;
  - обнуляет движение;
  - после окончания возвращает врага в `chase` или `idle`.

---

## Всплывающие цифры урона

### Общий renderer
- Новый общий файл: `res://scenes/ui/combat_text.gd`.
- Все цифры урона по врагам должны идти через `CombatText.show_damage(...)` или preload helper в конкретном скрипте.
- Настройка показа проверяется внутри renderer: если `SettingsManager.get_setting("damage_numbers") == false`, новые цифры не создаются.

### Обычный урон и криты
- Обычные попадания показывают число над врагом.
- Криты передают `{"is_crit": true}` и получают увеличенный текст + `!`.
- Крит-флаг прокинут через:
  - `res://scenes/skills/auto_arrow.gd`
  - `res://scenes/skills/arrow.gd`
  - `res://scenes/skills/fireball.gd`
  - `res://systems/abilities/abilities/slash_auto_attack.gd`
  - `res://systems/abilities/abilities/fireball_auto_attack.gd`

### Источники с иконками
Lightning, poison и thorns используют тот же renderer, но передают `source`:
- `{"source": "lightning"}` → значок молнии.
- `{"source": "poison"}` → значок яда.
- `{"source": "thorns"}` → значок отражения.

Основная интеграция находится в `res://systems/items/passive_component.gd`.

### Объединение burst damage
- Если несколько попаданий приходят почти одновременно рядом с одной целью, они объединяются в одну большую цифру.
- Окно объединения: `DAMAGE_MERGE_WINDOW = 0.10` секунды.
- Размер ячейки для объединения: `DAMAGE_MERGE_CELL_SIZE = 28.0` пикселей.
- Суммарная цифра становится крупнее и получает более толстый outline.
- Иконки источников сохраняются рядом с суммой, например молния + яд + число.

---

## Настройка показа цифр урона

### SettingsManager
- `res://systems/settings/settings_manager.gd`
- В `DEFAULTS` добавлено:
```gdscript
"damage_numbers": true
```

### SettingsScreen
- `res://scenes/ui/settings/settings_screen.gd`
- Во вкладке Controls добавлена строка:
  - `Цифры урона`
  - чекбокс `Показывать`
- Чекбокс пишет значение в `SettingsManager.set_setting("damage_numbers", v)`.
- Важно: строки настроек нужно править через Godot AI / UTF-8-aware workflow. Обычный shell/apply_patch уже приводил к битой кодировке русских строк в UI.

---

## Live update биндов

- `res://systems/settings/settings_manager.gd`
- После изменения или сброса бинда теперь эмитится `settings_changed`.
- Это нужно, чтобы UI биндов обновлялся, даже если игрок меняет клавиши уже во время запущенной игры.

---

## Важные технические правила после изменений

### Сигнатура take_damage
Все враги, наследующиеся от `res://systems/enemy/base_enemy.gd`, должны совпадать с родительской сигнатурой:
```gdscript
func take_damage(amount: float, knockback_dir: Vector2 = Vector2.ZERO, options: Dictionary = {}) -> void:
```

`options` используется для combat text (`is_crit`, `source`) и forced knockback (`force_hurt_knockback`, `knockback_strength`) и должен оставаться третьим аргументом. Для forced knockback caller передаёт направление в `knockback_dir`, а не заранее умноженный импульс; силу задаёт `knockback_strength`, дальше `BaseEnemy` масштабирует её через `EnemyData.knockback_force`.

### Helper для CombatText в BaseEnemy
- В `base_enemy.gd` helper называется `CombatTextHelper`.
- Не объявлять в наследнике константу с тем же именем, если она уже есть в родителе.
- Для boss override (`minotaur_enemy.gd`) можно использовать родительский `CombatTextHelper.show_damage(...)`.

### Не ломать boss behavior
- Босс может показывать цифры урона, но не должен получать stagger от обычного механизма.
- `minotaur_enemy.gd` сохраняет override `take_damage`, который не вызывает `relay_hit`.

---

# Изменения 16 мая 2026: тестовая комната, слоты, коллизии, оглушение

Основные файлы кода: `player.gd`, `world.gd`, `boss_test_room.gd`, `boss_dash_state.gd`, `ability_resource.gd`, `ability_component.gd`, `door.gd`, pickup-скрипты и projectile-скрипты.

---

## 1. Пятый активный слот навыка

У игрока 5 активных слотов навыков плюс отдельный survival-слот.

Актуальная раскладка действий для активных слотов:
```gdscript
slot_actions = ["skill_1", "skill_3", "skill_4", "skill_5", "skill_6"]
normal_ab_indices = [1, 2, 3, 4, 5]
```

Клавиши по умолчанию:
- `skill_1` -> клавиша `1`
- `skill_3` -> клавиша `2`
- `skill_4` -> клавиша `3`
- `skill_5` -> клавиша `4`
- `skill_6` -> клавиша `5`
- `skill_2` -> survival-слот, `Shift`

Важно: старое дублирование `skill_5` для 4-го и 5-го слота больше не актуально. Пятый слот должен использовать `skill_6`.

Затронутые места:
- `res://scenes/player/player.gd`
- `res://scenes/world/world.gd`
- `res://scenes/world/boss_test_room.gd`
- `res://systems/settings/settings_manager.gd`
- `res://project.godot`

---

## 2. boss_test_room должна соответствовать основной игре

`res://scenes/world/boss_test_room.gd` теперь должна вести себя как основная игра, а не как отдельный упрощённый режим.

Актуальные правила:
- Тестовая комната строит UI слотов навыков по тем же правилам, что и `world.gd`.
- Есть `AbilityRangeOverlay` для hover/targeting отображения радиусов навыков.
- Есть auto-слот, 5 активных слотов, survival-слот и locked passive-слот.
- Drag-n-drop активных слотов использует `_ability_order`, как в основной игре.
- Подбор предметов должен идти через те же правила inventory, passive replacement, upgrades и item panel refresh, что и в `world.gd`.
- Выбор персонажа в debug-панели респавнит игрока, а не просто меняет статы на старом теле.
- Кнопка `Респавн [R]` и клавиша `R` респавнят текущего игрока.
- Во время игры `F1` переводит в `res://scenes/world/boss_test_room.tscn`.
- По краям тестовой комнаты созданы непроходимые стены через `StaticBody2D`.

Важно: при респавне используется шаблон текущего `Player` из сцены, потому что отдельный `player.tscn` может не содержать весь набор дочерних узлов, ожидаемых боевой логикой.

---

## 3. Коллизии игрока, врагов и триггеров

Чтобы враги не прилипали к игроку и не двигали его вместе с собой, физические слои разделены.

Актуальная схема:
```gdscript
# player.gd
collision_layer = 1 << 1
collision_mask = 1 | (1 << 2)

# base_enemy.gd / minotaur_enemy.gd
collision_layer = 1 << 2
collision_mask = 1
```

Смысл:
- игрок находится на отдельном player body layer;
- игрок физически сталкивается со стенами и врагами;
- враги физически сталкиваются со стенами, но не толкают игрока своим body mask;
- игрок больше не должен проходить сквозь врагов;
- враги больше не должны прилипать к игроку и тащить его.

Побочный эффект этой схемы: `Area2D`-триггеры, которые раньше слушали только слой `1`, перестали видеть игрока. Поэтому для player-триггеров используется централизованная маска из `res://systems/collision_layers.gd`, которая видит старый и новый слой:
```gdscript
const PLAYER_TRIGGER_MASK := CollisionLayers.PLAYER_TRIGGER_MASK
```

Эта маска применяется в:
- `res://scenes/rooms/doors/door.gd`
- `res://scenes/rooms/room_item_pickup.gd`
- `res://scenes/rooms/shop_item_pickup.gd`
- `res://scenes/rooms/shop_refresh_pickup.gd`
- `res://scenes/gold_orb/gold_orb.gd`
- `res://scenes/xp_orb/xp_orb.gd`
- `res://scenes/enemies/enemy_arrow.gd`

Важно: если добавляется новый `Area2D`, который должен реагировать на игрока через `body_entered`, ему нужно явно ставить `collision_mask = CollisionLayers.PLAYER_TRIGGER_MASK`.

---

## 4. Двери

`Door` остаётся `Area2D`, который эмитит `player_entered(direction)` при входе игрока.

Актуальная логика:
- закрытая дверь отключает `CollisionShape2D`, поэтому не триггерится;
- открытая дверь включает `CollisionShape2D`;
- `collision_mask` двери должен видеть player body layer;
- из-за разделения слоёв игрока и врагов дверь теперь использует `CollisionLayers.PLAYER_TRIGGER_MASK`.

Если игрок визуально стоит в двери, но переход не происходит, сначала проверять маску двери и player layer.

---

## 5. Минотавр: чардж, стена, конус

`res://scenes/enemies/boss_dash_state.gd`:
- после окончания dash добавлена задержка `POST_DASH_CONE_DELAY = 0.1` перед переходом в `boss_cone`;
- это нужно, чтобы конусная атака не уходила в короткую неуязвимость игрока после удара/чарджа;
- при врезании минотавра в реальную стену включается фаза `wall_stun` на `WALL_STUN_DUR = 1.8`;
- во время `wall_stun` над минотавром показывается `StunIndicator` со звёздами;
- столкновения с `CharacterBody2D` не считаются стеной для wall-stun.

`minotaur_enemy.gd` по-прежнему переопределяет `stun()` от WarCry:
- WarCry не оглушает босса;
- вместо этого босс получает 50 урона и красный flash.

Важно: wall-stun минотавра и `stun()` от WarCry - разные механики.

---

## 6. Оглушение игрока после чарджа минотавра

Когда минотавр попадает dash-атакой по игроку:
- игрок получает урон;
- включается knockback;
- `knockback_active = true` считается состоянием оглушения игрока;
- над игроком показывается `StunIndicator` со звёздами на время `PLAYER_STUN_TIME`;
- после окончания `_KnockbackNode` снимает `knockback_active` и временную invincible-защиту.

В `player.gd` добавлен helper:
```gdscript
func is_stunned() -> bool:
	return knockback_active
```

Другие системы должны проверять именно `is_stunned()`, а не напрямую читать `knockback_active`, если им важно состояние стана.

---

## 7. StunIndicator

`res://systems/enemy/stun_indicator.gd` теперь используется не только обычными врагами, но и минотавром/игроком.

Актуальные поля:
```gdscript
var duration: float = 2.0
var enemy: Node = null
var offset_y: float = -28.0
```

`offset_y` нужен, чтобы поднимать звёзды на разную высоту для разных целей:
- обычные враги: дефолтная высота;
- игрок: чуть выше головы;
- минотавр: выше из-за большего спрайта.

Если `enemy` задан и у цели есть `_unstun()`, индикатор вызовет `_unstun()` по окончании. Для игрока и wall-stun минотавра `enemy` не задаётся: состояние снимается своей логикой таймера.

---

## 8. Использование способностей в оглушении

В `AbilityResource` добавлен флаг:
```gdscript
@export var can_use_while_stunned: bool = false
```

Правило:
- по умолчанию никакие способности нельзя использовать в стане;
- способность можно будет разрешить в стане только если явно поставить `can_use_while_stunned = true`;
- на момент добавления флага таких способностей нет.

`AbilityComponent` теперь проверяет этот флаг в одном центральном месте:
- `try_use_ability(index)` не запускает способность в стане, если флаг выключен;
- `use_all_auto()` также не запускает auto-abilities в стане, если флаг выключен;
- cooldown не тратится, потому что `AbilityResource.try_use()` не вызывается.

`player.gd` дополнительно блокирует UX-слой:
- не открывает targeting/aiming для запрещённой способности в стане;
- не подтверждает уже начатое прицеливание, если игрок оглушён;
- показывает floating text `Оглушение!` вместо `Откат!`.

При добавлении будущих исключений проверять:
```gdscript
ability.can_use_while_stunned = true
```
или выставлять это поле в `.tres`/ресурсе способности.

---

## 9. Проверки после изменений

После правок были smoke-запуски:
- `res://scenes/world/boss_test_room.tscn`
- `res://scenes/world/world.tscn`

Game/editor логи не показали runtime/parse ошибок.

Тестовый раннер проекта по-прежнему не используется, потому что `res://tests/` отсутствует.

---

## 10. Центрирование панели скиллов

`UI/SkillsBar` в `res://scenes/world/world.tscn` и runtime-layout в `res://scenes/world/world.gd` теперь используют `BoxContainer.ALIGNMENT_CENTER`.

Раньше панель была фиксированной ширины и прижимала слоты к правому краю (`ALIGNMENT_END`), из-за чего скиллы визуально съезжали относительно нижнего HUD.

---

## 11. Экран характеристик: здоровье, опыт, мультивыстрел

`res://scenes/ui/player_stats_screen.gd` теперь показывает:
- здоровье сердцами по правилу HUD (`20 HP = 1 сердце`, частично заполненное сердце отображается отдельным символом);
- опыт в формате `xp / xp_to_next`;
- мультивыстрел из актуального meta-ключа `extra_projectiles`, который пишет `item_upgrade_multishot.gd`.

Формат мультивыстрела: `+N (M всего)`, где `N` — дополнительные снаряды, `M` — базовый снаряд плюс бонус.

---

## 12. Пересчет уворота лучницы после изменения скорости

`ArcherPassive` получил общий helper:
```gdscript
ArcherPassive.refresh_on_player(player)
```

Он находит пассивку в `AbilityComponent` слота [7] и повторно применяет расчет `dodge_chance` от текущей `player.speed`.

Helper вызывается после известных изменений скорости:
- `ArcherSprintAbility` при старте и завершении спринта;
- `item_berserk_ring.gd`;
- `item_wind_boots.gd`;
- `_apply_saved_stats()` в `world.gd` после загрузки скорости и meta-статов.

Сохранение теперь пишет актуальный meta-ключ мультивыстрела `extra_projectiles`; при загрузке старый `multishot_count` мигрирует в `extra_projectiles` как fallback.

---

## 13. Единый откат в tooltip навыка и карточке skill-апгрейда

`ItemDatabase.get_skill_upgrade_description()` теперь принимает необязательный `world_context`.

`ItemPickupScreen` передаёт туда текущий world, поэтому карточка улучшения уже взятого SKILL может найти живой `AbilityResource` игрока и читать текущие `damage`/`cooldown_time` из него. Это убирает расхождение, когда tooltip навыка показывал откат с учётом предметов, а карточка апгрейда считала от свежего базового ресурса.

Fallback без `world_context` сохранён для тестов и изолированных вызовов: расчёт идёт по `AbilityResource.upgrade_to_level()`.

---

## 14. Rich tooltip для слотов навыков

Глобальный `Tooltip` теперь использует `RichTextLabel`, но BBCode включается только по флагу `tooltip_trigger.tooltip_use_bbcode`.

`AbilitySlot` вызывает `set_rich_tip()`:
- название навыка выделяется увеличенным жирным текстом и цветом;
- значение `Перезарядка` подсвечивается цветом;
- теги подсвечиваются отдельным цветом.

Обычные tooltip-ы предметов/пассивок остаются plain text, чтобы старые строки с квадратными скобками не интерпретировались как BBCode.

---

## 15. Feedback после выбора апгрейда

После выбора UPGRADE или повторного SKILL-апгрейда теперь появляется короткий toast:
- название предмета/навыка;
- новый уровень `Ур. X/Y`;
- строка изменения (`Урон`, `Откат`, `Снаряды`, `Шанс крита` и т.п.).

`ItemSlotsPanel.flash_item(script_path)` подсвечивает обновлённый слот на нижней панели. Поведение добавлено и в `world.gd`, и в `boss_test_room.gd`, чтобы тестовая комната не расходилась с основной игрой.

8 июня 2026: общий вызов toast/flash вынесен в `ItemRewardController.show_choice_feedback(...)`. `world.gd` и `boss_test_room.gd` больше не дублируют проверку `feedback_level` и подсветку слота; локально у них остаётся только отрисовка самого toast через `_show_item_upgrade_feedback(...)`.

Также в `ItemRewardController.replace_passive_item(...)` вынесена замена PASSIVE-предметов: обе сцены используют один путь для `on_remove`, обновления `inventory.passives`, `on_pickup` и `ItemSlotsPanel.replace_passive(...)`.

### Локализация pause menu
- `LocalizationManager` получил ключи `pause.*` для заголовка, кнопок и confirm-диалогов паузы.
- `pause_menu.gd` обновляет тексты через `LocalizationManager.loc(...)` при открытии сцены и при `language_changed`.
- Старые RU-хардкоды в UI проверены: player-facing pause menu перенесён в локализацию, debug/legacy UI (`boss_test_room`, `shop_screen.gd`, `item_choice_screen.gd`) можно оставлять без перевода до отдельного решения по dev tools.

### Spawn pools как таблицы данных
- Составы врагов вынесены из больших констант `base_room.gd` в `RoomSpawnPoolTable` data-resources.
- Текущие таблицы: `legacy_spawn_pool_table.gd` и `archer_spawn_pool_table.gd` под `res://systems/rooms/spawn_pools/`.
- `BaseRoom` теперь выбирает таблицу по `biome_id`, а для `dungeon_level >= 2` таблица использует `level_2_pools`.

### Вражеские ловушки
- Добавлен `res://scenes/rooms/enemy_trap.gd`: скрытая ловушка раскрывается рядом с игроком, срабатывает в малом радиусе, наносит урон и замедляет игрока.
- `BaseRoom` спавнит ловушки в NORMAL и SECRET/SECRET2 комнатах; позиции сохраняются в `room_data` meta `enemy_trap_positions`.
- `player.gd` получил `apply_slow(...)` / `get_slow_multiplier()`, чтобы player-side slow не менял базовый `speed` напрямую.

13 июня 2026:
- `BaseRoom` теперь восстанавливает оставшиеся ловушки при повторном входе в cleared-комнату из `room_data` meta `enemy_trap_positions`.
- `EnemyTrap` эмитит `triggered(local_position)`, а `BaseRoom` удаляет эту позицию из сохранённого списка, поэтому сработавшая ловушка не воскресает.
- `test_enemy_trap.gd` покрывает восстановление ловушек и удаление сработавшей позиции; полный `test_run` после правки проходит `89/89`.

14 июня 2026:
- `EnemyTrap` получил разоружение через input action `disarm_trap` (`E` по умолчанию). Если игрок находится в `disarm_radius`, нажатие запускает channel на `disarm_duration`: игрок удерживается на месте, над ним рисуется прогресс-бар.
- Если во время разоружения `health_changed` сообщает здоровье ниже стартового значения channel, разоружение отменяется. При успехе ловушка эмитит `disarmed(local_position)`, удаляется из `room_data` meta `enemy_trap_positions` и не появляется при повторном входе в комнату.
- На время разоружения `EnemyTrap` вызывает `player.block_ability_use("enemy_trap_disarm")`: автоатаки, слотовые навыки и запуск навыков из UI не срабатывают, cooldown не тратится. При отмене/успехе blocker снимается.
- `player.take_damage()` теперь возвращает bool: `false`, если урон отменён неуязвимостью или уворотом. `EnemyTrap` использует этот результат и не накладывает slow, если игрок уклонился от ловушки.
- Когда игрок находится в `disarm_radius`, но ещё не вошёл в `trigger_radius`, `EnemyTrap` показывает над ловушкой подсказку с текущей привязкой `disarm_trap`, например `E Взаимодействовать`; подсказка скрывается при старте разоружения или срабатывании.
- `SettingsManager` добавляет `disarm_trap` в runtime InputMap и список переназначаемых действий как «Взаимодействовать».
- `ItemSlotsPanel` после сборки, добавления предметов и внешнего `_layout_ui()` подгоняет ширину под viewport и центрирует себя, чтобы при восстановлении сейва разросшаяся панель не уезжала вправо.
- `SkillsBar` теперь раскладывается через прямые `offset_left/right` и повторно подгоняется после пересборки слотов. Это чинит реальный save/load-сценарий, где бар получал правильный X, но сохранял старую большую ширину, из-за чего иконки навыков центрировались далеко справа.
- `BaseRoom` создаёт `BoundaryColliders` на слое `WORLD`: стены процедурных комнат теперь физически непроходимы, а открытые двери оставляют проходы. Коллизии перестраиваются при `setup()` и `open_doors()`.
- Полный `test_run` после правки проходит `104/104`.

### Self-cast навыки без подтверждения
- `AbilityResource` получил `requires_targeting()`: по умолчанию навык требует targeting только если `get_range_info()` не пустой.
- `player.gd` в TARGETING_CONFIRM/TARGETING_QUICK применяет no-target навыки сразу, без locked overlay и повторного подтверждения.
- Shift лучницы (`ArcherSprintAbility`) теперь срабатывает сразу в любом режиме таргетинга; прицельные навыки вроде `ArrowFanAbility` сохраняют старый confirm/quick flow.

### Player buff/debuff HUD
- Добавлен `res://scenes/ui/player_status_bar.gd`: компактная полоса статусов игрока рядом с сердцами в `world.gd` и `boss_test_room.gd`.
- `player.gd` получил `get_player_status_effects()`, а player-side slow отдаёт `duration`/`remaining` для HUD.
- `ArcherSprintAbility` пишет runtime meta таймера, поэтому Shift лучницы отображается как бафф с убывающим заполнением.

### Прогрессия dungeon level
- 13 июня 2026: максимальный dungeon level поднят до 5 (`world.gd`, `MAX_DUNGEON_LEVEL`).
- `RoomSpawnPoolTable.get_pools()` для этажей 2+ использует `level_2_pools`, но больше не умножает весь пул врагов. Количество растёт небольшим добавочным бонусом: примерно +1/+2 врага за следующий этаж, распределённо по непустым типам врагов.
- `BaseRoom` при спавне врага дублирует `EnemyData` и умножает `max_health`/`xp_reward` на `2^(dungeon_level - 1)`, поэтому HP удваивается после каждого уровня без изменения shared enemy resources.
- Портал входа/выхода между уровнями закрыт отдельной правкой ниже: boss-room больше не переводит на следующий этаж мгновенно.

### Фиксированные слоты магазина
- 13 июня 2026: позиции shop item pickup теперь считаются по фиксированной трёхслотовой сетке, а кнопка `ShopRefresh` всегда ставится в 4-й слот (`SHOP_ITEM_SLOT_COUNT`).
- При повторном входе в магазин после покупок и после refresh кнопка обновления больше не сжимается к оставшимся товарам и не появляется на позиции предмета.

### Отравленная стрела лучницы
- 13 июня 2026: добавлен приобретаемый SKILL «Отравленная стрела [Лучница]»: `res://systems/abilities/abilities/poison_arrow_ability.gd`, `res://systems/items/definitions/skills/item_skill_poison_arrow.gd`, регистрация в `ItemDatabase`.
- Навык стреляет `res://scenes/skills/poison_arrow_projectile.tscn` по курсору, использует `ray` overlay на 260px, наносит 18 прямого урона цели и при попадании взрывается в радиусе 82px.
- Взрыв накладывает на врагов `PoisonDoT`: 6 урона раз в секунду 4.0с. Повторное наложение refresh-ит существующий `PoisonDoT`, поэтому enemy HP bar показывает один убывающий poison-статус.
- Через `$imagegen` добавлены sprite-sheet стрелы `res://assets/generated/poison_arrow_projectile_sheet.png` и sprite-sheet взрыва `res://assets/generated/poison_arrow_burst_sheet.png`; используются существующие иконки `icon_poison_arrow.png` и `items/icon_item_skill_poison_arrow.png`.
- Добавлен smoke-тест `test_poison_arrow_ability.gd`; полный `test_run` после правки проходит `84/84`.

### Порталы между dungeon levels
- 13 июня 2026: добавлен `res://scenes/rooms/level_portal.gd`. Портал использует `CollisionLayers.PLAYER_TRIGGER_MASK`, рисуется кодом и армится как дверь: если игрок уже стоит в зоне при появлении, переход не срабатывает до выхода и повторного входа.
- `world.gd` спавнит входной портал в центре стартовой комнаты каждого этажа. После зачистки boss-room на этажах 1-4 игра больше не переходит дальше мгновенно, а создаёт выходной портал в центре комнаты; вход в него запускает следующий dungeon level. После босса 5-го этажа остаётся текущий victory flow.
- Через `$Image Gen` добавлен 4-кадровый pixel-art sheet портала `res://assets/generated/level_portal_sheet.png`, нарезанный из исходного chroma-key изображения. `LevelPortal` грузит PNG через `Image.load()`, создаёт `Sprite2D` с `hframes=4`, кадром 96×112 и `z_index=-1`, чтобы арка была чуть выше персонажа и рисовалась за ним.
- Добавлен smoke-тест `test_level_portal.gd`; полный `test_run` после правки проходит `90/90`.

### Единый gameplay entrypoint
- 14 июня 2026: добавлен `res://scenes/world/world_gameplay_host.tscn` со скриптом `WorldGameplayHost`.
- Чтобы запустить текущий полный gameplay из `world.tscn` внутри другой сцены, нужно добавить в неё `WorldGameplayHost`. По умолчанию он инстанцирует `res://scenes/world/world.tscn` дочерним узлом `World`, поэтому новые сцены не должны вручную копировать `Entities/Player`, `RoomManager`, `UI`, pause/item/victory экраны и `MusicPlayer`.
- API хоста: `start_gameplay()`, `stop_gameplay()`, `get_gameplay_root()`. Ограничение: это запуск всего `world.tscn` целиком; для кастомной структуры нужно задавать другой `gameplay_scene` или расширять сам `world.tscn`.
- Добавлен smoke-тест `test_world_gameplay_host.gd`; полный `test_run` после правки проходит `93/93`.
- `WorldGameplayHost` получил authored-room режим: `use_authored_room`, `authored_room_path_from_gameplay`, `authored_room_type`, `authored_room_starts_cleared`, `open_authored_doors_on_start`. В этом режиме `world.gd` пропускает `MapGenerator`/`RoomManager.load_room()`, а `RoomManager.adopt_existing_room()` подключает уже существующий room node и открывает его authored-двери без вызова `setup()`.
- `test_world_gameplay_host.gd` покрывает прокидывание authored-room настроек; полный `test_run` после правки проходит `94/94`.

### Засады в authored-комнатах
- 14 июня 2026: добавлен `res://scenes/rooms/ambush_trigger.tscn` / `ambush_trigger.gd`.
- `AmbushTrigger` — Area2D-объект: при входе игрока выбирает один словарь из `spawn_pools` и спавнит врагов в пределах `spawn_area_size`. Поддерживаемые enemy keys: `goblin`, `goblin_archer`, `wolf`, `orc`, `orc_warrior`, `archer`, `bomber`, `firefly_spirit`, `summoner`.
- Спавн идёт в `spawn_parent_path` (`../Enemies` по умолчанию), позиции проверяют world collision layer, чтобы не выбирать точку внутри физических стен/тел мира.
- Сигналы: `ambush_started(spawned_enemies)` и `ambush_cleared`.
- Добавлен smoke-тест `test_ambush_trigger.gd`; полный `test_run` после правки проходит `96/96`.

15 июня 2026:
- `Door` получил export `unlock_after_cleared_paths`: дверь может ссылаться на `AmbushTrigger`/спавнер и оставаться закрытой, пока все источники не очистятся. Если открытие уже было запрошено, дверь откроется сама после сигнала `ambush_cleared`/`spawner_cleared`/`all_enemies_dead`/`cleared`.
- `AmbushTrigger` теперь хранит `is_cleared()`, а `forest_nswe_room.tscn` связывает свою дверь с `../AmbushTrigger` и запускается в authored-room режиме. `RoomManager` теперь находит двери не только в `Doors/*`, но и прямыми детьми authored-комнаты; smoke-тест `room_manager_routes` покрывает этот обход.
- `InteractionMode` упрощён до двух глобальных режимов: `Касание` и `Кнопка взаимодействия`. `Door`, `LevelPortal`, `room_item_pickup`, `shop_item_pickup` и `heart_pickup` сохраняют export `interaction_mode` (`Use Settings`, `Touch`, `Button`), `Use Settings` берёт режим из меню `Настройки -> Управление -> Взаимодействие с объектами`; дефолт глобальной настройки — `Касание`.
- В режиме `Кнопка взаимодействия` двери, порталы, предметы, магазинные предметы и сердца показывают trap-style подсказку с текущей привязкой общего action `disarm_trap`; пикапы показывают её только в радиусе подбора, а не на дальнем proximity.