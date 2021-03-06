
1. Högerklicka på Windows Store-app-projekt i Visual Studio Solution Explorer. Välj sedan **Store** > **associera appen med butiken**.

    ![Associera appen med Windows Store](./media/app-service-mobile-register-wns/notification-hub-associate-win8-app.png)
2. I guiden väljer du **nästa**. Logga sedan in med ditt Microsoft-konto. I **reservera ett nytt namn för appen**, Skriv ett namn för din app och markera sedan **reservera**.
3. När registreringen app har skapats, Välj ny app. Välj **nästa**, och välj sedan **associera**. Den här processen lägger till nödvändig information för registrering av Windows Store i programmanifestet.
4. Upprepa steg 1 och 3 för Windows Phone Store-app-projekt med samma registrering som du skapade tidigare för Windows Store-app.  
5. Gå till den [Windows Dev Center](https://dev.windows.com/en-us/overview), och logga sedan in med ditt Microsoft-konto. I **Mina appar**, Välj den nya app-registreringen. Expandera **Services** > **Push-meddelanden**.
6. På den **Push-meddelanden** sidan under **Windows Push Notification Services (WNS) och Microsoft Azure Mobile Apps**väljer **webbplatsen Live-tjänster**.  Anteckna värdena för den **paket-SID** och *aktuella* värde i **Programhemlighet**. 

    ![Appinställningen i developer center](./media/app-service-mobile-register-wns/mobile-services-win8-app-push-auth.png)

   > [!IMPORTANT]
   > Programhemligheten och paket-SID:et är viktiga säkerhetsuppgifter. Inte dela dessa värden med någon eller distribuera dem med din app.
   >
   >
