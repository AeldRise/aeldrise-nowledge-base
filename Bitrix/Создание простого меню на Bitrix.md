# Создание простого меню на Bitrix

## Файл `template.php`
```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();

if (empty($arResult)) return;
?>

<nav class="dropdown-nav">
    <ul class="dropdown-nav__list">
        <?php foreach ($arResult as $item): ?>
            <?php
            $itemClass = 'dropdown-nav__item';
            $linkClass = 'dropdown-nav__link';

            if ($item['SELECTED']) {
                $itemClass .= ' dropdown-nav__item--active';
                $linkClass .= ' dropdown-nav__link--active';
            }

            if (!empty($item['CHILDREN'])) {
                $itemClass .= ' dropdown-nav__item--has-children';
            }
            ?>

            <li class="<?= $itemClass ?>">
                <a href="<?= $item['LINK'] ?>" class="<?= $linkClass ?>">
                    <?= $item['TEXT'] ?>
                    <?php if (!empty($item['CHILDREN'])): ?>
                        <span class="dropdown-nav__arrow"></span>
                    <?php endif; ?>
                </a>

                <?php if (!empty($item['CHILDREN'])): ?>
                    <div class="dropdown-nav__submenu">
                        <ul class="dropdown-nav__sublist">
                            <?php foreach ($item['CHILDREN'] as $child): ?>
                                <?php
                                $subitemClass = 'dropdown-nav__subitem';
                                $sublinkClass = 'dropdown-nav__sublink';

                                if ($child['SELECTED']) {
                                    $subitemClass .= ' dropdown-nav__subitem--active';
                                    $sublinkClass .= ' dropdown-nav__sublink--active';
                                }

                                if (!empty($child['CHILDREN'])) {
                                    $subitemClass .= ' dropdown-nav__subitem--has-children';
                                }
                                ?>

                                <li class="<?= $subitemClass ?>">
                                    <a href="<?= $child['LINK'] ?>" class="<?= $sublinkClass ?>">
                                        <?= $child['TEXT'] ?>
                                        <?php if (!empty($child['CHILDREN'])): ?>
                                            <span class="dropdown-nav__subarrow"></span>
                                        <?php endif; ?>
                                    </a>

                                    <?php if (!empty($child['CHILDREN'])): ?>
                                        <div class="dropdown-nav__subsubmenu">
                                            <ul class="dropdown-nav__subsublist">
                                                <?php foreach ($child['CHILDREN'] as $subchild): ?>
                                                    <?php
                                                    $subsubitemClass = 'dropdown-nav__subsubitem';
                                                    $subsublinkClass = 'dropdown-nav__subsublink';

                                                    if ($subchild['SELECTED']) {
                                                        $subsubitemClass .= ' dropdown-nav__subsubitem--active';
                                                        $subsublinkClass .= ' dropdown-nav__subsublink--active';
                                                    }
                                                    ?>

                                                    <li class="<?= $subsubitemClass ?>">
                                                        <a href="<?= $subchild['LINK'] ?>" class="<?= $subsublinkClass ?>">
                                                            <?= $subchild['TEXT'] ?>
                                                        </a>
                                                    </li>
                                                <?php endforeach; ?>
                                            </ul>
                                        </div>
                                    <?php endif; ?>
                                </li>
                            <?php endforeach; ?>
                        </ul>
                    </div>
                <?php endif; ?>
            </li>
        <?php endforeach; ?>
    </ul>
</nav>
```
## Файл `style.css`
```css
/* Простое пурпурное меню */
.dropdown-nav {
    background: #8a2be2;
    font-family: Arial;
}

.dropdown-nav__list {
    margin: 0;
    padding: 0;
    list-style: none;
    display: flex;
}

.dropdown-nav__item {
    position: relative;
}

.dropdown-nav__item:hover > .dropdown-nav__submenu {
    display: block;
}

/* Активные элементы */
.dropdown-nav__item--active {
    background: #9932cc;
}

.dropdown-nav__link--active {
    background: #9932cc;
    color: white;
    font-weight: bold;
}

.dropdown-nav__subitem--active {
    background: #f5f0ff;
    border-left: 3px solid #8a2be2;
}

.dropdown-nav__subsublink--active {
    background: #f5f0ff;
    color: #8a2be2;
    font-weight: bold;
}

/* Ссылки */
.dropdown-nav__link {
    display: block;
    padding: 15px 20px;
    color: white;
    text-decoration: none;
}

.dropdown-nav__link:hover {
    background: #9932cc;
}

/* Выпадающее меню */
.dropdown-nav__submenu {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    min-width: 200px;
    background: white;
    border: 1px solid #d8bfd8;
    box-shadow: 0 2px 5px rgba(138, 43, 226, 0.2);
    z-index: 100;
}

.dropdown-nav__sublist {
    margin: 0;
    padding: 10px 0;
    list-style: none;
}

.dropdown-nav__sublink {
    display: block;
    padding: 10px 15px;
    color: #4b0082;
    text-decoration: none;
}

.dropdown-nav__sublink:hover {
    background: #f5f0ff;
    color: #8a2be2;
}

/* Под-подменю */
.dropdown-nav__subsubmenu {
    display: none;
    position: absolute;
    top: 0;
    left: 100%;
    min-width: 200px;
    background: white;
    border: 1px solid #d8bfd8;
    box-shadow: 0 2px 5px rgba(138, 43, 226, 0.2);
}

.dropdown-nav__subitem:hover > .dropdown-nav__subsubmenu {
    display: block;
}

.dropdown-nav__subsublink {
    display: block;
    padding: 10px 15px;
    color: #841844;
    text-decoration: none;
}

.dropdown-nav__subsublink:hover {
    background: #f5f0ff;
    color: #841844;
}

/* Мобильная версия */
@media (max-width: 768px) {
    .dropdown-nav__list {
        flex-direction: column;
    }

    .dropdown-nav__submenu,
    .dropdown-nav__subsubmenu {
        position: static;
        box-shadow: none;
        border: none;
        border-left: 2px solid #d8bfd8;
        margin-left: 20px;
    }
}
```
## Файл result_modifier.php
```php
<?
if(!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true)die();

$menuList = array();
$lev = 0;
$lastInd = 0;
$parents = array();

foreach ($arResult as $arItem) {
    $lev = $arItem['DEPTH_LEVEL'];

    if ($arItem['IS_PARENT']) {
        $arItem['CHILDREN'] = array();
    }

    if ($lev == 1) {
        $menuList[] = $arItem;
        $lastInd = count($menuList)-1;
        $parents[$lev] = &$menuList[$lastInd];
    } else {
        $parents[$lev-1]['CHILDREN'][] = $arItem;
        $lastInd = count($parents[$lev-1]['CHILDREN'])-1;
        $parents[$lev] = &$parents[$lev-1]['CHILDREN'][$lastInd];
    }
}

// Подсвечиваем родительские элементы активных пунктов
foreach ($menuList as &$item) {
    // Проверяем детей первого уровня
    if (!empty($item['CHILDREN'])) {
        $hasActiveChild = false;

        foreach ($item['CHILDREN'] as &$child) {
            // Проверяем внуков
            if (!empty($child['CHILDREN'])) {
                $hasActiveGrandChild = false;

                foreach ($child['CHILDREN'] as &$grandchild) {
                    if ($grandchild['SELECTED']) {
                        $hasActiveGrandChild = true;
                        $child['SELECTED'] = true; // Помечаем родителя как активный
                        break;
                    }
                }
                unset($grandchild);

                if ($hasActiveGrandChild) {
                    $hasActiveChild = true;
                }
            }

            if ($child['SELECTED']) {
                $hasActiveChild = true;
            }
        }
        unset($child);

        // Если есть активный ребенок, помечаем родителя как активный
        if ($hasActiveChild && !$item['SELECTED']) {
            $item['SELECTED'] = true;
        }
    }
}
unset($item);

$arResult = $menuList;
?>
```