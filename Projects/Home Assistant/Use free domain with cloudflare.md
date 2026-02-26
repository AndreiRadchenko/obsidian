---
date_created: 2026-02-25
date_modified: 2026-02-25
document_type: note
project: Home Assistant
tags: home-assistant note
---
[[Projects/Home Assistant/Home|Home]] | [[Projects/Home Assistant/Meetings/All Meetings|Meetings]] | [[Projects/Home Assistant/Notes/All Notes|Notes]] | [[Projects/Home Assistant/References|References]]
# Use free domain with cloudflare
**Overview**
Description:: 

## Note

# **Налаштовуємо віддалений доступ до Home Assistant через Cloudflare**


---

## **1. Реєстрація домену**

1. Зареєструйте домен (наприклад, безкоштовний у nic.ua).
    
2. Домен стане вашою зовнішньою адресою для Home Assistant.
    
3. Після активації домену ви побачите зелену крапку в особистому кабінеті.
    
4. Домен безкоштовний назавжди, але щороку його потрібно продовжувати. 
    

---

## **2. Налаштування DNS у Cloudflare**

1. У особистому кабінеті Cloudflare перейдіть до NS-серверів домену.
    
2. Змініть їх на:
    
    - dina.ns.cloudflare.com
        
    - hal.ns.cloudflare.com 
        
    

---

## **3. Додавання сайту в Cloudflare**

1. Створіть обліковий запис на cloudflare.com.
    
2. У розділі **Websites** натисніть **Add a Site** і додайте домен.
    
3. Після додавання статус має бути **Active**. 
    

---

## **4. Видалення SSL-сертифікатів**

  

> Перед використанням Cloudflared потрібно видалити всі інші додатки, що обробляють SSL, наприклад:

- > DuckDN
    
- > Let’s Encrypt
    

  

Це необхідно для уникнення конфліктів. 

---

## **5. Додавання Cloudflared у Home Assistant**

1. У Home Assistant перейдіть до **Конфігурація → Додатки → Магазин доповнень**.
    
2. Зверху праворуч натисніть три крапки → **Репозиторії**.
    
3. Додайте адресу:
    

```
https://github.com/brenner-tobias/ha-addons
```

3.   
    
4. Перезавантажте магазин доповнень.
    
5. Знайдіть і встановіть **Cloudflared**.
    
6. Увімкніть опції _Запуск після увімкнення_ та _Watchdog_. 
    

---

## **6. Налаштування Cloudflared**

  

Перед запуском додатку додайте в configuration.yaml такі рядки:

```
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 172.30.33.0/24
    - 127.0.0.1
```

Це потрібно для правильної роботи проксування. 

---

## **7. Запуск і авторизація тунелю**

1. Запустіть додаток **Cloudflared**.
    
2. Перейдіть у вкладку **Журнал** і натискайте **Оновити**, поки не з’явиться посилання для авторизації.
    
3. Відкрийте посилання у браузері.
    
4. Авторизуйтеся через акаунт Cloudflare. 
    

  

Після цього доступ до вашого Home Assistant має бути доступний через домен.

---

## **8. Перевірка доступу**

  

Після успішної авторизації:

- Введіть домен у браузері.
    
- Ви повинні побачити сторінку входу Home Assistant.
    
- Це також дозволить використовувати функції віддаленого доступу. 
    

  

> Якщо сторінка відкривається через HTTP без шифрування, налаштуйте автоматичне перенаправлення на https:

  

У **Cloudflare → SSL/TLS → Edge Certificates**:

- Увімкніть **Always Use HTTPS**. 
    

---

## **📌 Поради безпеки**

  

> Не використовуйте прості паролі для облікового запису Cloudflare або Home Assistant — це може поставити вашу систему під ризик. 

---

## References


---
[[Projects/Home Assistant/Home|Home]] | [[Projects/Home Assistant/Meetings/All Meetings|Meetings]] | [[Projects/Home Assistant/Notes/All Notes|Notes]] | [[Projects/Home Assistant/References|References]]
