# Создание простого меню на Bitrix

## Файл `template.php`
```php
<?php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) die();

if (empty($arResult)) return;
?>

<nav class="simple-dropdown">
    <ul class="simple-dropdown__list">
        <?php foreach ($arResult as $item): ?>
            <li class="simple-dropdown__item">
                <a href="<?= $item['LINK'] ?>" class="simple-dropdown__link">
                    <?= $item['TEXT'] ?>
                    <?php if (!empty($item['CHILDREN'])): ?>
                        <span class="simple-dropdown__arrow">▼</span>
                    <?php endif; ?>
                </a>

                <?php if (!empty($item['CHILDREN'])): ?>
                    <ul class="simple-dropdown__submenu">
                        <?php foreach ($item['CHILDREN'] as $child): ?>
                            <li class="simple-dropdown__subitem">
                                <a href="<?= $child['LINK'] ?>" class="simple-dropdown__sublink">
                                    <?= $child['TEXT'] ?>
                                    <?php if (!empty($child['CHILDREN'])): ?>
                                        <span class="simple-dropdown__subarrow">▶</span>
                                    <?php endif; ?>
                                </a>

                                <?php if (!empty($child['CHILDREN'])): ?>
                                    <ul class="simple-dropdown__subsubmenu">
                                        <?php foreach ($child['CHILDREN'] as $subchild): ?>
                                            <li class="simple-dropdown__subsubitem">
                                                <a href="<?= $subchild['LINK'] ?>" class="simple-dropdown__subsublink">
                                                    <?= $subchild['TEXT'] ?>
                                                </a>
                                            </li>
                                        <?php endforeach; ?>
                                    </ul>
                                <?php endif; ?>
                            </li>
                        <?php endforeach; ?>
                    </ul>
                <?php endif; ?>
            </li>
        <?php endforeach; ?>
    </ul>
</nav>
```
## Файл `style.css`
```css
.simple-dropdown {
    background: #333;
    font-family: Arial, sans-serif;
}

.simple-dropdown__list {
    display: flex;
    margin: 0;
    padding: 0;
    list-style: none;
}

.simple-dropdown__item {
    position: relative;
}

.simple-dropdown__item:hover > .simple-dropdown__submenu {
    display: block;
}

/* Ссылки первого уровня */
.simple-dropdown__link {
    display: block;
    padding: 15px 20px;
    color: white;
    text-decoration: none;
    background: #333;
}

.simple-dropdown__link:hover {
    background: #555;
}

.simple-dropdown__arrow {
    margin-left: 5px;
    font-size: 10px;
}

/* Выпадающее меню */
.simple-dropdown__submenu {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    min-width: 200px;
    margin: 0;
    padding: 0;
    list-style: none;
    background: white;
    border: 1px solid #ddd;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
    z-index: 100;
}

.simple-dropdown__subitem {
    position: relative;
}

.simple-dropdown__subitem:hover > .simple-dropdown__subsubmenu {
    display: block;
}

/* Ссылки второго уровня */
.simple-dropdown__sublink {
    display: block;
    padding: 10px 15px;
    color: #333;
    text-decoration: none;
    border-bottom: 1px solid #eee;
}

.simple-dropdown__sublink:hover {
    background: #f5f5f5;
}

.simple-dropdown__subarrow {
    float: right;
    font-size: 10px;
}

/* Подменю третьего уровня */
.simple-dropdown__subsubmenu {
    display: none;
    position: absolute;
    top: 0;
    left: 100%;
    min-width: 200px;
    margin: 0;
    padding: 0;
    list-style: none;
    background: white;
    border: 1px solid #ddd;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.simple-dropdown__subsubitem {
    display: block;
}

.simple-dropdown__subsublink {
    display: block;
    padding: 10px 15px;
    color: #333;
    text-decoration: none;
    border-bottom: 1px solid #eee;
}

.simple-dropdown__subsublink:hover {
    background: #f5f5f5;
}

/* Адаптивность */
@media (max-width: 768px) {
    .simple-dropdown__list {
        flex-direction: column;
    }

    .simple-dropdown__submenu {
        position: static;
        box-shadow: none;
        border: none;
        border-left: 2px solid #ddd;
        margin-left: 20px;
    }

    .simple-dropdown__subsubmenu {
        position: static;
        box-shadow: none;
        border: none;
        border-left: 2px solid #ddd;
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
$arResult = $menuList;
```