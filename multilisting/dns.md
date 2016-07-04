# Имеющиеся конфиги (внутри /usr/local/Reductor)

## userinfo/hooks/custom_bindings_dns

Формат:

- id списка
- id ipset к которому применяется правило, "-" для применения ко всем абонентам
- ip для редиректа, "-" для белых списков
- режим матчинга: 
  - exact - точный матчинг
  - with_subdomains - будет блокироваться домен и все его субдомены
- комментарий (англоязычный комментарий для того, чтобы при просмотре правил файрвола всё было понятно).

```
1 3 10.50.140.73 exact black exact_blacklist
2 3 - exact white exact_whitelist
3 3 10.50.140.73 with_subdomains black subdomains_blacklist
```

## userinfo/hooks/custom_ipsets/3 
```
10.30.31.2
```

## userinfo/hooks/custom_domains/1 
```
blog.denypage.ru
acidkernel.com
badsite.com
www.somesite.com
```

## userinfo/hooks/custom_domains/2
```
www.carbonsoft.ru
```

## userinfo/hooks/custom_domains/3 
```
carbonsoft.ru
```

# Результаты применения этих списков

Для пользователя с IP адресом 10.30.31.2:

- blog.denypage.ru будет заблокирован
- denypage.ru будет доступен
- acidkernel.com будет заблокирован
- blog.acidkernel.com не будет заблокирован
- badsite.com будет заблокирован
- www.badsite.com не будет заблокирован
- www.somesite.com будет заблокирован
- somesite.com не будет заблокирован
- www.carbonsoft.ru не будет заблокирован
- carbonsoft.ru будет заблокирован
- любой субдомен carbonsoft.ru, например docs.carbonsoft.ru будет заблокирован

# Побочные эффекты

Не рекомендуется использовать дополнительные списки для крупных списков абонентов, поскольку каждое правило значительно увеличивает нагрузку на сервер фильтрации, независимо (ладно, с очень слабой зависимостью) от числа доменов в фильтруемом списке, поскольку в каждом правиле приходится разбирать пакет заново.
