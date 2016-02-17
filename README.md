# sssssssssss
Automatically exported from code.google.com/p/sssssssssss

... ... @@ -0,0 +1,2 @@ 
 1   +all:  
 2   +	zip -r optout.xpi *  
 


 

2 2 additions & 0 deletions  optoutplugin/firefox/chrome.manifest  


... ... @@ -0,0 +1,2 @@ 
 1   +content optout  chrome/content/  
 2   +overlay chrome://browser/content/browser.xul chrome://optout/content/optout.xul  
 


 

 BIN   Binary file added  optoutplugin/firefox/chrome/content/google.png  


 





Viewer requires iframe. 


 

88 88 additions & 0 deletions  optoutplugin/firefox/chrome/content/optout.js  


... ... @@ -0,0 +1,88 @@ 
 1   +/*  
 2   + * Copyright 2009 Google Inc.  
 3   + *  
 4   + * Licensed under the Apache License, Version 2.0 (the "License");  
 5   + * you may not use this file except in compliance with the License.  
 6   + * You may obtain a copy of the License at  
 7   + *  
 8   + *     http://www.apache.org/licenses/LICENSE-2.0  
 9   + *  
 10   + * Unless required by applicable law or agreed to in writing, software  
 11   + * distributed under the License is distributed on an "AS IS" BASIS,  
 12   + * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 13   + * See the License for the specific language governing permissions and  
 14   + * limitations under the License.  
 15   + */  
 16   +  
 17   +/**  
 18   + * @fileoverview Firefox plugin to save advertising opt-out  
 19   + * preference permanently.  
 20   + * @author vali@google.com (Valentin Gheorghita)  
 21   + */  
 22   +  
 23   +/**  
 24   + * Check if third party cookies are disabled.  
 25   + * @return {boolean} True if they are disabled.  
 26   + */  
 27   +function AdvertisingOptOut_ThirdPartyCookiesDisabled() {  
 28   +  return Components.classes['@mozilla.org/preferences-service;1'].  
 29   +      getService(Components.interfaces.nsIPrefBranch).  
 30   +      getIntPref('network.cookie.cookieBehavior') != 0;  
 31   +}  
 32   +  
 33   +/**  
 34   + * Check if the user should be opt out, and in affirmative case the cookie  
 35   + * is installed.  
 36   + */  
 37   +function AdvertisingOptOut_OptOut() {  
 38   +  if (!AdvertisingOptOut_ThirdPartyCookiesDisabled()) {  
 39   +    var ios = Components.classes['@mozilla.org/network/io-service;1'].  
 40   +        getService(Components.interfaces.nsIIOService);  
 41   +    var prefService = Components.classes['@mozilla.org/preferences-service;1'].  
 42   +        getService(Components.interfaces.nsIPrefService);  
 43   +    var prefOptOut = prefService.getBranch('extensions.advertisingoptout.');  
 44   +    var cookieUri = ios.newURI(  
 45   +        prefOptOut.getCharPref('google.url'), null, null);  
 46   +    var cookieSvc = Components.classes['@mozilla.org/cookieService;1'].  
 47   +        getService(Components.interfaces.nsICookieService);  
 48   +    cookieSvc.setCookieString(cookieUri, null,  
 49   +                              prefOptOut.getCharPref('google.cookie'), null);  
 50   +  }  
 51   +}  
 52   +  
 53   +/**  
 54   + * Listener variable for cookie deleted.  
 55   + */  
 56   +var AdvertisingOptOut_CookieListener =  
 57   +{  
 58   +  observe: function(subject, topic, data) {  
 59   +    if (topic == 'cookie-changed') {  
 60   +      if (data == 'cleared' || data == 'deleted') {  
 61   +        AdvertisingOptOut_OptOut();  
 62   +      }  
 63   +    }  
 64   +  }  
 65   +};  
 66   +  
 67   +/**  
 68   + * Load the extension.  
 69   + */  
 70   +function AdvertisingOptOut_Load() {  
 71   +  Components.classes['@mozilla.org/observer-service;1'].  
 72   +      getService(Components.interfaces.nsIObserverService).  
 73   +      addObserver(AdvertisingOptOut_CookieListener, 'cookie-changed', false);  
 74   +  var period = Components.classes['@mozilla.org/preferences-service;1'].  
 75   +      getService(Components.interfaces.nsIPrefBranch).  
 76   +      getIntPref('extensions.advertisingoptout.checkPeriodInS');  
 77   +  window.setInterval(AdvertisingOptOut_OptOut, period * 1000);  
 78   +  AdvertisingOptOut_OptOut();  
 79   +}  
 80   +  
 81   +/**  
 82   + * Unload the extension.  
 83   + */  
 84   +function AdvertisingOptOut_Unload() {  
 85   +  Components.classes['@mozilla.org/observer-service;1'].  
 86   +      getService(Components.interfaces.nsIObserverService).  
 87   +      removeObserver(AdvertisingOptOut_CookieListener, 'cookie-changed');  
 88   +}  
 


 

13 13 additions & 0 deletions  optoutplugin/firefox/chrome/content/optout.xul  


... ... @@ -0,0 +1,13 @@ 
 1   +<?xml version="1.0"?>  
 2   +<overlay id="optout"  
 3   +         xmlns="http://www.mozilla.org/keymaster/gatekeeper/there.is.only.xul">  
 4   +  
 5   +  <script type="application/x-javascript" src="chrome://optout/content/optout.js" />  
 6   +  
 7   +  <window id="main-window">  
 8   +    <script type="application/x-javascript">  
 9   +        window.addEventListener("load", AdvertisingOptOut_Load, false);  
 10   +        window.addEventListener("unload", AdvertisingOptOut_Unload, false);  
 11   +    </script>  
 12   +  </window>  
 13   +</overlay>  
 


 

19 19 additions & 0 deletions  optoutplugin/firefox/defaults/preferences/preferences.js  


... ... @@ -0,0 +1,19 @@ 
 1   +/*  
 2   + * Copyright 2009 Google Inc.  
 3   + *  
 4   + * Licensed under the Apache License, Version 2.0 (the "License");  
 5   + * you may not use this file except in compliance with the License.  
 6   + * You may obtain a copy of the License at  
 7   + *  
 8   + *     http://www.apache.org/licenses/LICENSE-2.0  
 9   + *  
 10   + * Unless required by applicable law or agreed to in writing, software  
 11   + * distributed under the License is distributed on an "AS IS" BASIS,  
 12   + * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 13   + * See the License for the specific language governing permissions and  
 14   + * limitations under the License.  
 15   + */  
 16   +pref('extensions.advertisingoptout.checkPeriodInS', 600);  
 17   +pref('extensions.advertisingoptout.google.url', 'http://doubleclick.net/');  
 18   +pref('extensions.advertisingoptout.google.cookie',  
 19   +     'id=OPT_OUT;domain=.doubleclick.net;expires=10 November 2030 00:59:06;');  
 


 

75 75 additions & 0 deletions  optoutplugin/firefox/install.rdf  


... ... @@ -0,0 +1,75 @@ 
 1   +<?xml version="1.0"?>  
 2   +  
 3   +<RDF xmlns="http://www.w3.org/1999/02/22-rdf-syntax-ns#"  
 4   +     xmlns:em="http://www.mozilla.org/2004/em-rdf#">  
 5   +  
 6   +  <Description about="urn:mozilla:install-manifest">  
 7   +    <em:id>optout@google.com</em:id>  
 8   +    <em:version>1.0</em:version>  
 9   +    <em:type>2</em:type>  
 10   +    <!-- Target Application this extension can install into,  
 11   +         with minimum and maximum supported versions. -->  
 12   +    <em:targetApplication>  
 13   +      <Description>  
 14   +        <em:id>{ec8030f7-c20a-464f-9b0e-13a3a9e97384}</em:id>  
 15   +        <em:minVersion>1.5</em:minVersion>  
 16   +        <em:maxVersion>3.0.*</em:maxVersion>  
 17   +      </Description>  
 18   +    </em:targetApplication>  
 19   +    <!-- Front End MetaData -->  
 20   +    <em:name>Advertising Cookie Opt-out</em:name>  
 21   +    <em:description>Save your opt-out preference permanently.</em:description>  
 22   +    <em:iconURL>chrome://optout/content/google.png</em:iconURL>  
 23   +    <em:creator>Google Inc.</em:creator>  
 24   +    <em:contributor>Valentin Gheorghita</em:contributor>  
 25   +    <em:homepageURL>http://www.google.com/ads/preferences/plugin/opt-out.html</em:homepageURL>  
 26   +    <em:updateURL>https://dl-ssl.google.com/optoutplugin/google-optout-plugin-update.rdf</em:updateURL>  
 27   +    <em:localized>  
 28   +      <Description>  
 29   +        <em:locale>de</em:locale>  
 30   +        <em:name>Deaktivieren des Cookies für Anzeigenvorgaben</em:name>  
 31   +        <em:description>Dauerhaftes Deaktivieren des DoubleClick-Cookies. Dieses Cookie wird von Google als Cookie für Anzeigenvorgaben verwendet.</em:description>  
 32   +      </Description>  
 33   +      <Description>  
 34   +        <em:locale>da</em:locale>  
 35   +        <em:name>Fravalg af annonceringscookie</em:name>  
 36   +        <em:description>Permanent fravalg af DoubleClick-cookien, der er en annonceringscookie, Google bruger.</em:description>  
 37   +      </Description>  
 38   +      <Description>  
 39   +        <em:locale>it</em:locale>  
 40   +        <em:name>Disattivazione del cookie per la pubblicità</em:name>  
 41   +        <em:description>Disattiva in modo permanente il cookie DoubleClick, un cookie per la pubblicità utilizzato da Google.</em:description>  
 42   +      </Description>  
 43   +      <Description>  
 44   +        <em:locale>no</em:locale>  
 45   +        <em:name>Bortvalg av informasjonskapsel for annonsering</em:name>  
 46   +        <em:description>Velg bort DoubleClick-informasjonskapselen, som er en informasjonskapsel for annonsering som Google bruker, permanent.</em:description>  
 47   +      </Description>  
 48   +      <Description>  
 49   +        <em:locale>fi</em:locale>  
 50   +        <em:name>Mainosevästeen käytöstäpoisto</em:name>  
 51   +        <em:description>Poista pysyvästi DoubleClick-eväste, joka on Googlen käyttämä mainoseväste.</em:description>  
 52   +      </Description>  
 53   +      <Description>  
 54   +        <em:locale>fr</em:locale>  
 55   +        <em:name>Désactivation du cookie publicitaire</em:name>  
 56   +        <em:description>Désactive de façon permanente le cookie DoubleClick, qui est un cookie publicitaire utilisé par Google.</em:description>  
 57   +      </Description>  
 58   +      <Description>  
 59   +        <em:locale>sv</em:locale>  
 60   +        <em:name>Välja bort annonscookies</em:name>  
 61   +        <em:description>Välj permanent bort DoubleClick-cookien, som är en annonscookie som Google använder.</em:description>  
 62   +      </Description>  
 63   +      <Description>  
 64   +        <em:locale>es</em:locale>  
 65   +        <em:name>Inhabilitación de cookies de publicidad</em:name>  
 66   +        <em:description>Inhabilite la cookie de DoubleClick de forma permanente. Se trata de una cookie de publicidad utilizada por Google.</em:description>  
 67   +      </Description>  
 68   +      <Description>  
 69   +        <em:locale>nl</em:locale>  
 70   +        <em:name>Afmelden voor advertentiecookie</em:name>  
 71   +        <em:description>Permanent afmelden voor de DoubleClick-cookie. Dit is een advertentiecookie die door Google wordt gebruikt.</em:description>  
 72   +      </Description>  
 73   +    <em:localized>  
 74   +  </Description>  
 75   +</RDF>  
 


 

191 191 additions & 0 deletions  optoutplugin/ie/COPYING  


... ... @@ -0,0 +1,191 @@ 
 1   +                                 Apache License  
 2   +                           Version 2.0, January 2004  
 3   +                        http://www.apache.org/licenses/  
 4   +  
 5   +   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION  
 6   +  
 7   +   1. Definitions.  
 8   +  
 9   +      "License" shall mean the terms and conditions for use, reproduction,  
 10   +      and distribution as defined by Sections 1 through 9 of this document.  
 11   +  
 12   +      "Licensor" shall mean the copyright owner or entity authorized by  
 13   +      the copyright owner that is granting the License.  
 14   +  
 15   +      "Legal Entity" shall mean the union of the acting entity and all  
 16   +      other entities that control, are controlled by, or are under common  
 17   +      control with that entity. For the purposes of this definition,  
 18   +      "control" means (i) the power, direct or indirect, to cause the  
 19   +      direction or management of such entity, whether by contract or  
 20   +      otherwise, or (ii) ownership of fifty percent (50%) or more of the  
 21   +      outstanding shares, or (iii) beneficial ownership of such entity.  
 22   +  
 23   +      "You" (or "Your") shall mean an individual or Legal Entity  
 24   +      exercising permissions granted by this License.  
 25   +  
 26   +      "Source" form shall mean the preferred form for making modifications,  
 27   +      including but not limited to software source code, documentation  
 28   +      source, and configuration files.  
 29   +  
 30   +      "Object" form shall mean any form resulting from mechanical  
 31   +      transformation or translation of a Source form, including but  
 32   +      not limited to compiled object code, generated documentation,  
 33   +      and conversions to other media types.  
 34   +  
 35   +      "Work" shall mean the work of authorship, whether in Source or  
 36   +      Object form, made available under the License, as indicated by a  
 37   +      copyright notice that is included in or attached to the work  
 38   +      (an example is provided in the Appendix below).  
 39   +  
 40   +      "Derivative Works" shall mean any work, whether in Source or Object  
 41   +      form, that is based on (or derived from) the Work and for which the  
 42   +      editorial revisions, annotations, elaborations, or other modifications  
 43   +      represent, as a whole, an original work of authorship. For the purposes  
 44   +      of this License, Derivative Works shall not include works that remain  
 45   +      separable from, or merely link (or bind by name) to the interfaces of,  
 46   +      the Work and Derivative Works thereof.  
 47   +  
 48   +      "Contribution" shall mean any work of authorship, including  
 49   +      the original version of the Work and any modifications or additions  
 50   +      to that Work or Derivative Works thereof, that is intentionally  
 51   +      submitted to Licensor for inclusion in the Work by the copyright owner  
 52   +      or by an individual or Legal Entity authorized to submit on behalf of  
 53   +      the copyright owner. For the purposes of this definition, "submitted"  
 54   +      means any form of electronic, verbal, or written communication sent  
 55   +      to the Licensor or its representatives, including but not limited to  
 56   +      communication on electronic mailing lists, source code control systems,  
 57   +      and issue tracking systems that are managed by, or on behalf of, the  
 58   +      Licensor for the purpose of discussing and improving the Work, but  
 59   +      excluding communication that is conspicuously marked or otherwise  
 60   +      designated in writing by the copyright owner as "Not a Contribution."  
 61   +  
 62   +      "Contributor" shall mean Licensor and any individual or Legal Entity  
 63   +      on behalf of whom a Contribution has been received by Licensor and  
 64   +      subsequently incorporated within the Work.  
 65   +  
 66   +   2. Grant of Copyright License. Subject to the terms and conditions of  
 67   +      this License, each Contributor hereby grants to You a perpetual,  
 68   +      worldwide, non-exclusive, no-charge, royalty-free, irrevocable  
 69   +      copyright license to reproduce, prepare Derivative Works of,  
 70   +      publicly display, publicly perform, sublicense, and distribute the  
 71   +      Work and such Derivative Works in Source or Object form.  
 72   +  
 73   +   3. Grant of Patent License. Subject to the terms and conditions of  
 74   +      this License, each Contributor hereby grants to You a perpetual,  
 75   +      worldwide, non-exclusive, no-charge, royalty-free, irrevocable  
 76   +      (except as stated in this section) patent license to make, have made,  
 77   +      use, offer to sell, sell, import, and otherwise transfer the Work,  
 78   +      where such license applies only to those patent claims licensable  
 79   +      by such Contributor that are necessarily infringed by their  
 80   +      Contribution(s) alone or by combination of their Contribution(s)  
 81   +      with the Work to which such Contribution(s) was submitted. If You  
 82   +      institute patent litigation against any entity (including a  
 83   +      cross-claim or counterclaim in a lawsuit) alleging that the Work  
 84   +      or a Contribution incorporated within the Work constitutes direct  
 85   +      or contributory patent infringement, then any patent licenses  
 86   +      granted to You under this License for that Work shall terminate  
 87   +      as of the date such litigation is filed.  
 88   +  
 89   +   4. Redistribution. You may reproduce and distribute copies of the  
 90   +      Work or Derivative Works thereof in any medium, with or without  
 91   +      modifications, and in Source or Object form, provided that You  
 92   +      meet the following conditions:  
 93   +  
 94   +      (a) You must give any other recipients of the Work or  
 95   +          Derivative Works a copy of this License; and  
 96   +  
 97   +      (b) You must cause any modified files to carry prominent notices  
 98   +          stating that You changed the files; and  
 99   +  
 100   +      (c) You must retain, in the Source form of any Derivative Works  
 101   +          that You distribute, all copyright, patent, trademark, and  
 102   +          attribution notices from the Source form of the Work,  
 103   +          excluding those notices that do not pertain to any part of  
 104   +          the Derivative Works; and  
 105   +  
 106   +      (d) If the Work includes a "NOTICE" text file as part of its  
 107   +          distribution, then any Derivative Works that You distribute must  
 108   +          include a readable copy of the attribution notices contained  
 109   +          within such NOTICE file, excluding those notices that do not  
 110   +          pertain to any part of the Derivative Works, in at least one  
 111   +          of the following places: within a NOTICE text file distributed  
 112   +          as part of the Derivative Works; within the Source form or  
 113   +          documentation, if provided along with the Derivative Works; or,  
 114   +          within a display generated by the Derivative Works, if and  
 115   +          wherever such third-party notices normally appear. The contents  
 116   +          of the NOTICE file are for informational purposes only and  
 117   +          do not modify the License. You may add Your own attribution  
 118   +          notices within Derivative Works that You distribute, alongside  
 119   +          or as an addendum to the NOTICE text from the Work, provided  
 120   +          that such additional attribution notices cannot be construed  
 121   +          as modifying the License.  
 122   +  
 123   +      You may add Your own copyright statement to Your modifications and  
 124   +      may provide additional or different license terms and conditions  
 125   +      for use, reproduction, or distribution of Your modifications, or  
 126   +      for any such Derivative Works as a whole, provided Your use,  
 127   +      reproduction, and distribution of the Work otherwise complies with  
 128   +      the conditions stated in this License.  
 129   +  
 130   +   5. Submission of Contributions. Unless You explicitly state otherwise,  
 131   +      any Contribution intentionally submitted for inclusion in the Work  
 132   +      by You to the Licensor shall be under the terms and conditions of  
 133   +      this License, without any additional terms or conditions.  
 134   +      Notwithstanding the above, nothing herein shall supersede or modify  
 135   +      the terms of any separate license agreement you may have executed  
 136   +      with Licensor regarding such Contributions.  
 137   +  
 138   +   6. Trademarks. This License does not grant permission to use the trade  
 139   +      names, trademarks, service marks, or product names of the Licensor,  
 140   +      except as required for reasonable and customary use in describing the  
 141   +      origin of the Work and reproducing the content of the NOTICE file.  
 142   +  
 143   +   7. Disclaimer of Warranty. Unless required by applicable law or  
 144   +      agreed to in writing, Licensor provides the Work (and each  
 145   +      Contributor provides its Contributions) on an "AS IS" BASIS,  
 146   +      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or  
 147   +      implied, including, without limitation, any warranties or conditions  
 148   +      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A  
 149   +      PARTICULAR PURPOSE. You are solely responsible for determining the  
 150   +      appropriateness of using or redistributing the Work and assume any  
 151   +      risks associated with Your exercise of permissions under this License.  
 152   +  
 153   +   8. Limitation of Liability. In no event and under no legal theory,  
 154   +      whether in tort (including negligence), contract, or otherwise,  
 155   +      unless required by applicable law (such as deliberate and grossly  
 156   +      negligent acts) or agreed to in writing, shall any Contributor be  
 157   +      liable to You for damages, including any direct, indirect, special,  
 158   +      incidental, or consequential damages of any character arising as a  
 159   +      result of this License or out of the use or inability to use the  
 160   +      Work (including but not limited to damages for loss of goodwill,  
 161   +      work stoppage, computer failure or malfunction, or any and all  
 162   +      other commercial damages or losses), even if such Contributor  
 163   +      has been advised of the possibility of such damages.  
 164   +  
 165   +   9. Accepting Warranty or Additional Liability. While redistributing  
 166   +      the Work or Derivative Works thereof, You may choose to offer,  
 167   +      and charge a fee for, acceptance of support, warranty, indemnity,  
 168   +      or other liability obligations and/or rights consistent with this  
 169   +      License. However, in accepting such obligations, You may act only  
 170   +      on Your own behalf and on Your sole responsibility, not on behalf  
 171   +      of any other Contributor, and only if You agree to indemnify,  
 172   +      defend, and hold each Contributor harmless for any liability  
 173   +      incurred by, or claims asserted against, such Contributor by reason  
 174   +      of your accepting any such warranty or additional liability.  
 175   +  
 176   +   END OF TERMS AND CONDITIONS  
 177   +  
 178   +  
 179   +   Copyright 2009 Google Inc.  
 180   +  
 181   +   Licensed under the Apache License, Version 2.0 (the "License");  
 182   +   you may not use this file except in compliance with the License.  
 183   +   You may obtain a copy of the License at  
 184   +  
 185   +       http://www.apache.org/licenses/LICENSE-2.0  
 186   +  
 187   +   Unless required by applicable law or agreed to in writing, software  
 188   +   distributed under the License is distributed on an "AS IS" BASIS,  
 189   +   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 190   +   See the License for the specific language governing permissions and  
 191   +   limitations under the License.  
 


 

81 81 additions & 0 deletions  optoutplugin/ie/bho.cc  


... ... @@ -0,0 +1,81 @@ 
 1   +// Copyright 2009 Google Inc.  
 2   +//  
 3   +// Licensed under the Apache License, Version 2.0 (the "License");  
 4   +// you may not use this file except in compliance with the License.  
 5   +// You may obtain a copy of the License at  
 6   +//  
 7   +//      http://www.apache.org/licenses/LICENSE-2.0  
 8   +//  
 9   +// Unless required by applicable law or agreed to in writing, software  
 10   +// distributed under the License is distributed on an "AS IS" BASIS,  
 11   +// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +// See the License for the specific language governing permissions and  
 13   +// limitations under the License.  
 14   +  
 15   +#include <atlstr.h>  
 16   +#include <time.h>  
 17   +#include <wininet.h>  
 18   +#include "bho.h"  
 19   +  
 20   +  
 21   +// Sets the doubleclick.net cookie to id=OPT_OUT. The expiration time is the  
 22   +// current time, to which delta_days is added.  
 23   +// Use a negative delta_days to delete the cookie.  
 24   +void SetCookie(int delta_days) {  
 25   +  time_t expiration = time(NULL) + delta_days * 3600 * 24;  
 26   +  struct tm timeinfo;  
 27   +  gmtime_s(&timeinfo, &expiration);  
 28   +  
 29   +  const int kBufferSize = 50;  
 30   +  wchar_t date_buffer[kBufferSize];  
 31   +  wcsftime(date_buffer, kBufferSize, L"%a,%d-%b-%Y %X GMT", &timeinfo);  
 32   +  
 33   +  ATL::CString cookie_data;  
 34   +  cookie_data.Format(L"id = OPT_OUT; expires = %s", date_buffer);  
 35   +  
 36   +  InternetSetCookie(L"http://doubleclick.net", NULL, cookie_data);  
 37   +}  
 38   +  
 39   +  
 40   +// Implementation of IObjectWithSiteImpl::SetSite.  
 41   +STDMETHODIMP CPersistentOptOutBHO::SetSite(IUnknown* site) {  
 42   +  if (site != NULL) {  
 43   +    HRESULT hr = site->QueryInterface(IID_IWebBrowser2,  
 44   +                                      reinterpret_cast<void **>(&web_browser_));  
 45   +    if (SUCCEEDED(hr)) {  
 46   +      hr = DispEventAdvise(web_browser_);  
 47   +      advised_ = true;  
 48   +    }  
 49   +  } else {  // site == NULL  
 50   +    if (advised_) {  
 51   +      DispEventUnadvise(web_browser_);  
 52   +      advised_ = false;  
 53   +    }  
 54   +    web_browser_.Release();  
 55   +  }  
 56   +  return IObjectWithSiteImpl<CPersistentOptOutBHO>::SetSite(site);  
 57   +}  
 58   +  
 59   +// If enabled, refreshes the opt-out cookie before a page is loaded.  
 60   +// This only applies to top-level documents (not to frames).  
 61   +void STDMETHODCALLTYPE CPersistentOptOutBHO::BeforeNavigate(  
 62   +    IDispatch *disp,  
 63   +    VARIANT *url,  
 64   +    VARIANT *flags,  
 65   +    VARIANT *target_frame_name,  
 66   +    VARIANT *post_data,  
 67   +    VARIANT *headers,  
 68   +    VARIANT_BOOL *cancel) {  
 69   +  if (web_browser_ != NULL && disp != NULL) {  
 70   +    ATL::CComPtr<IUnknown> unknown1;  
 71   +    ATL::CComPtr<IUnknown> unknown2;  
 72   +    if (SUCCEEDED(web_browser_->QueryInterface(  
 73   +                      IID_IUnknown, reinterpret_cast<void**>(&unknown1))) &&  
 74   +        SUCCEEDED(disp->QueryInterface(  
 75   +                      IID_IUnknown, reinterpret_cast<void**>(&unknown2)))) {  
 76   +      if (unknown1 == unknown2) {  
 77   +        SetCookie(90);  // Expires in 3 months.  
 78   +      }  
 79   +    }  
 80   +  }  
 81   +}  
 


 

67 67 additions & 0 deletions  optoutplugin/ie/bho.h  


... ... @@ -0,0 +1,67 @@ 
 1   +// Copyright 2009 Google Inc.  
 2   +//  
 3   +// Licensed under the Apache License, Version 2.0 (the "License");  
 4   +// you may not use this file except in compliance with the License.  
 5   +// You may obtain a copy of the License at  
 6   +//  
 7   +//      http://www.apache.org/licenses/LICENSE-2.0  
 8   +//  
 9   +// Unless required by applicable law or agreed to in writing, software  
 10   +// distributed under the License is distributed on an "AS IS" BASIS,  
 11   +// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +// See the License for the specific language governing permissions and  
 13   +// limitations under the License.  
 14   +  
 15   +#pragma once  
 16   +  
 17   +#include <atlbase.h>  
 18   +#include <atlcom.h>  
 19   +#include <atlctl.h>  
 20   +#include <shlguid.h>  
 21   +#include <exdispid.h>  
 22   +#include <shlobj.h>  
 23   +#include "opt_out.h"  // Generated by the MIDL compiler.  
 24   +#include "resource.h"  
 25   +  
 26   +// Once loaded, this BHO ensures that the doubleclick cookie is persistently set  
 27   +// to id=OPT_OUT, by resetting it every time a new page is loaded.  
 28   +class ATL_NO_VTABLE CPersistentOptOutBHO  
 29   +    : public ATL::CComObjectRootEx<ATL::CComSingleThreadModel>,  
 30   +      public ATL::CComCoClass<CPersistentOptOutBHO, &CLSID_PersistentOptOutBHO>,  
 31   +      public ATL::IObjectWithSiteImpl<CPersistentOptOutBHO>,  
 32   +      public ATL::IDispatchImpl<IPersistentOptOutBHO, &IID_IPersistentOptOutBHO,  
 33   +                                &LIBID_PersistentOptOutLib, 1, 0>,  
 34   +      public ATL::IDispEventImpl<1, CPersistentOptOutBHO,  
 35   +                                 &DIID_DWebBrowserEvents2,  
 36   +                                 &LIBID_SHDocVw, 1, 1> {  
 37   + public:  
 38   +  CPersistentOptOutBHO() { }  
 39   +  
 40   +  STDMETHOD(SetSite)(IUnknown *site);  
 41   +  
 42   +  DECLARE_REGISTRY_RESOURCEID(IDR_PERSISTENTOPTOUTBHO)  
 43   +  
 44   +  BEGIN_COM_MAP(CPersistentOptOutBHO)  
 45   +    COM_INTERFACE_ENTRY(IPersistentOptOutBHO)  
 46   +    COM_INTERFACE_ENTRY(IDispatch)  
 47   +    COM_INTERFACE_ENTRY(IObjectWithSite)  
 48   +  END_COM_MAP()  
 49   +  
 50   +  BEGIN_SINK_MAP(CPersistentOptOutBHO)  
 51   +    SINK_ENTRY_EX(1, DIID_DWebBrowserEvents2,  
 52   +                  DISPID_BEFORENAVIGATE2, BeforeNavigate)  
 53   +  END_SINK_MAP()  
 54   +  
 55   +  void STDMETHODCALLTYPE BeforeNavigate(IDispatch *disp,  
 56   +                                        VARIANT *url,  
 57   +                                        VARIANT *flags,  
 58   +                                        VARIANT *target_frame_name,  
 59   +                                        VARIANT *post_data,  
 60   +                                        VARIANT *headers,  
 61   +                                        VARIANT_BOOL *cancel);  
 62   + private:  
 63   +  ATL::CComPtr<IWebBrowser2> web_browser_;  
 64   +  bool advised_;  
 65   +};  
 66   +  
 67   +OBJECT_ENTRY_AUTO(__uuidof(PersistentOptOutBHO), CPersistentOptOutBHO)  
 


 

38 38 additions & 0 deletions  optoutplugin/ie/bho.rgs  


... ... @@ -0,0 +1,38 @@ 
 1   +HKCR {  
 2   +  opt_out.PersistentOptOutBHO.1 = s 'PersistentOptOutBHO Class' {  
 3   +    CLSID = s '{8E425EB4-ADBD-4816-B1E8-49BB9DECF034}'  
 4   +  }  
 5   +  opt_out.PersistentOptOutBHO = s 'PersistentOptOutBHO Class' {  
 6   +    CLSID = s '{8E425EB4-ADBD-4816-B1E8-49BB9DECF034}'  
 7   +    CurVer = s 'opt_out.PersistentOptOutBHO.1'  
 8   +  }  
 9   +  NoRemove CLSID {  
 10   +    ForceRemove {8E425EB4-ADBD-4816-B1E8-49BB9DECF034} = s 'Adverstising Cookie Opt-out' {  
 11   +      ProgID = s 'opt_out.PersistentOptOutBHO.1'  
 12   +      VersionIndependentProgID = s 'opt_out.PersistentOptOutBHO'  
 13   +      ForceRemove 'Programmable'  
 14   +      InprocServer32 = s '%MODULE%' {  
 15   +        val ThreadingModel = s 'Apartment'  
 16   +      }  
 17   +      'TypeLib' = s '{F240BFEF-88DD-4F6D-9816-D32E509B929B}'  
 18   +    }  
 19   +  }  
 20   +}  
 21   +  
 22   +HKLM {  
 23   +  NoRemove SOFTWARE {  
 24   +    NoRemove Microsoft {  
 25   +      NoRemove Windows {  
 26   +        NoRemove CurrentVersion {  
 27   +          NoRemove Explorer {  
 28   +            NoRemove 'Browser Helper Objects' {  
 29   +              ForceRemove '{8E425EB4-ADBD-4816-B1E8-49BB9DECF034}' = s 'Adverstising Cookie Opt-out' {  
 30   +                val 'NoExplorer' = d '1'  
 31   +              }  
 32   +            }  
 33   +          }  
 34   +        }  
 35   +      }  
 36   +    }  
 37   +  }  
 38   +}  
 


 

57 57 additions & 0 deletions  optoutplugin/ie/opt_out.cc  


... ... @@ -0,0 +1,57 @@ 
 1   +// Copyright 2009 Google Inc.  
 2   +//  
 3   +// Licensed under the Apache License, Version 2.0 (the "License");  
 4   +// you may not use this file except in compliance with the License.  
 5   +// You may obtain a copy of the License at  
 6   +//  
 7   +//      http://www.apache.org/licenses/LICENSE-2.0  
 8   +//  
 9   +// Unless required by applicable law or agreed to in writing, software  
 10   +// distributed under the License is distributed on an "AS IS" BASIS,  
 11   +// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +// See the License for the specific language governing permissions and  
 13   +// limitations under the License.  
 14   +  
 15   +  
 16   +// Implementation of DLL Exports.  
 17   +  
 18   +#include <atlbase.h>  
 19   +#include "resource.h"  
 20   +#include "opt_out.h"  // Generated by the MIDL compiler.  
 21   +  
 22   +class CPersistentOptOutModule  
 23   +    : public ATL::CAtlDllModuleT<CPersistentOptOutModule> {  
 24   + public:  
 25   +  DECLARE_LIBID(LIBID_PersistentOptOutLib)  
 26   +  DECLARE_REGISTRY_APPID_RESOURCEID(IDR_PERSISTENTOPTOUT,  
 27   +                                    "{3AA15E6E-F324-4714-9806-B9F7C9FDB2F8}")  
 28   +};  
 29   +  
 30   +CPersistentOptOutModule _AtlModule;  
 31   +  
 32   +// DLL Entry Point.  
 33   +extern "C" BOOL WINAPI DllMain(HINSTANCE instance,  
 34   +                               DWORD reason,  
 35   +                               LPVOID reserved) {  
 36   +  return _AtlModule.DllMain(reason, reserved);  
 37   +}  
 38   +  
 39   +// Used to determine whether the DLL can be unloaded by OLE.  
 40   +STDAPI DllCanUnloadNow(void) {  
 41   +  return _AtlModule.DllCanUnloadNow();  
 42   +}  
 43   +  
 44   +// Returns a class factory to create an object of the requested type.  
 45   +STDAPI DllGetClassObject(REFCLSID rclsid, REFIID riid, LPVOID* ppv) {  
 46   +  return _AtlModule.DllGetClassObject(rclsid, riid, ppv);  
 47   +}  
 48   +  
 49   +// Adds entries to the system registry.  
 50   +STDAPI DllRegisterServer(void) {  
 51   +  return _AtlModule.DllRegisterServer();  
 52   +}  
 53   +  
 54   +// Removes entries from the system registry.  
 55   +STDAPI DllUnregisterServer(void) {  
 56   +  return _AtlModule.DllUnregisterServer();  
 57   +}  
 


 

24 24 additions & 0 deletions  optoutplugin/ie/opt_out.def  


... ... @@ -0,0 +1,24 @@ 
 1   +; Copyright 2009 Google Inc.  
 2   +;  
 3   +; Licensed under the Apache License, Version 2.0 (the "License");  
 4   +; you may not use this file except in compliance with the License.  
 5   +; You may obtain a copy of the License at  
 6   +;  
 7   +;      http://www.apache.org/licenses/LICENSE-2.0  
 8   +;  
 9   +; Unless required by applicable law or agreed to in writing, software  
 10   +; distributed under the License is distributed on an "AS IS" BASIS,  
 11   +; WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +; See the License for the specific language governing permissions and  
 13   +; limitations under the License.  
 14   +  
 15   +  
 16   +; Declares the module parameters.  
 17   +  
 18   +LIBRARY      "opt_out.dll"  
 19   +  
 20   +EXPORTS  
 21   +  DllCanUnloadNow      PRIVATE  
 22   +  DllGetClassObject    PRIVATE  
 23   +  DllRegisterServer    PRIVATE  
 24   +  DllUnregisterServer  PRIVATE  
 


 

47 47 additions & 0 deletions  optoutplugin/ie/opt_out.idl  


... ... @@ -0,0 +1,47 @@ 
 1   +// Copyright 2009 Google Inc.  
 2   +//  
 3   +// Licensed under the Apache License, Version 2.0 (the "License");  
 4   +// you may not use this file except in compliance with the License.  
 5   +// You may obtain a copy of the License at  
 6   +//  
 7   +//      http://www.apache.org/licenses/LICENSE-2.0  
 8   +//  
 9   +// Unless required by applicable law or agreed to in writing, software  
 10   +// distributed under the License is distributed on an "AS IS" BASIS,  
 11   +// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +// See the License for the specific language governing permissions and  
 13   +// limitations under the License.  
 14   +  
 15   +  
 16   +// This file will be processed by the MIDL tool to produce the type library  
 17   +// and marshalling code.  
 18   +  
 19   +import "oaidl.idl";  
 20   +import "ocidl.idl";  
 21   +  
 22   +[  
 23   +  object,  
 24   +  uuid(B0608AA2-3C43-424B-9814-5A9DFF06AABE),  
 25   +  dual,  
 26   +  nonextensible,  
 27   +  helpstring("IPersistentOptOutBHO Interface"),  
 28   +  pointer_default(unique)  
 29   +]  
 30   +interface IPersistentOptOutBHO : IDispatch{ };  
 31   +  
 32   +[  
 33   +  uuid(F240BFEF-88DD-4F6D-9816-D32E509B929B),  
 34   +  version(1.0),  
 35   +  helpstring("PersistentOptOut 1.0 Type Library")  
 36   +]  
 37   +library PersistentOptOutLib  
 38   +{  
 39   +  importlib("stdole2.tlb");  
 40   +  [  
 41   +    uuid(8E425EB4-ADBD-4816-B1E8-49BB9DECF034),  
 42   +    helpstring("PersistentOptOutBHO Class")  
 43   +  ]  
 44   +  coclass PersistentOptOutBHO {  
 45   +    [default] interface IPersistentOptOutBHO;  
 46   +  };  
 47   +};  
 


 

61 61 additions & 0 deletions  optoutplugin/ie/opt_out.rc  


... ... @@ -0,0 +1,61 @@ 
 1   +// Copyright 2009 Google Inc.  
 2   +//  
 3   +// Licensed under the Apache License, Version 2.0 (the "License");  
 4   +// you may not use this file except in compliance with the License.  
 5   +// You may obtain a copy of the License at  
 6   +//  
 7   +//      http://www.apache.org/licenses/LICENSE-2.0  
 8   +//  
 9   +// Unless required by applicable law or agreed to in writing, software  
 10   +// distributed under the License is distributed on an "AS IS" BASIS,  
 11   +// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +// See the License for the specific language governing permissions and  
 13   +// limitations under the License.  
 14   +  
 15   +  
 16   +#include "resource.h"  
 17   +#include "winres.h"  
 18   +  
 19   +  
 20   +VS_VERSION_INFO VERSIONINFO  
 21   + FILEVERSION VERSION_MAJOR,VERSION_MINOR,VERSION_BUILD,VERSION_PATCH  
 22   + PRODUCTVERSION VERSION_MAJOR,VERSION_MINOR,VERSION_BUILD,VERSION_PATCH  
 23   + FILEFLAGSMASK 0x3fL  
 24   +#ifdef _DEBUG  
 25   + FILEFLAGS 0x1L  
 26   +#else  
 27   + FILEFLAGS 0x0L  
 28   +#endif  
 29   + FILEOS 0x4L  
 30   + FILETYPE 0x2L  
 31   + FILESUBTYPE 0x0L  
 32   +BEGIN  
 33   +    BLOCK "StringFileInfo"  
 34   +    BEGIN  
 35   +        BLOCK "040904e4"  
 36   +        BEGIN  
 37   +            VALUE "CompanyName", "Google Inc"  
 38   +            VALUE "FileVersion", VERSION_NUMBER  
 39   +            VALUE "InternalName", "opt_out.dll"  
 40   +            VALUE "OriginalFilename", "opt_out.dll"  
 41   +            VALUE "ProductName", "Advertising Cookie Opt-out"  
 42   +            VALUE "ProductVersion", VERSION_NUMBER  
 43   +        END  
 44   +    END  
 45   +    BLOCK "VarFileInfo"  
 46   +    BEGIN  
 47   +        VALUE "Translation", 0x409, 1252  
 48   +    END  
 49   +END  
 50   +  
 51   +// Registry  
 52   +IDR_PERSISTENTOPTOUT              REGISTRY              "opt_out.rgs"  
 53   +IDR_PERSISTENTOPTOUTBHO           REGISTRY              "bho.rgs"  
 54   +  
 55   +// String table  
 56   +STRINGTABLE  
 57   +BEGIN  
 58   +    IDS_PROJNAME            "opt_out"  
 59   +END  
 60   +  
 61   +1	TYPELIB "opt_out.tlb"  No newline at end of file  
 


 

8 8 additions & 0 deletions  optoutplugin/ie/opt_out.rgs  


... ... @@ -0,0 +1,8 @@ 
 1   +HKCR {  
 2   +  NoRemove AppID {  
 3   +    '%APPID%' = s 'opt_out'  
 4   +    'opt_out.dll' {  
 5   +      val AppID = s '%APPID%'  
 6   +    }  
 7   +  }  
 8   +}  
 


 

20 20 additions & 0 deletions  optoutplugin/ie/resource.h  


... ... @@ -0,0 +1,20 @@ 
 1   +// Copyright 2009 Google Inc.  
 2   +//  
 3   +// Licensed under the Apache License, Version 2.0 (the "License");  
 4   +// you may not use this file except in compliance with the License.  
 5   +// You may obtain a copy of the License at  
 6   +//  
 7   +//      http://www.apache.org/licenses/LICENSE-2.0  
 8   +//  
 9   +// Unless required by applicable law or agreed to in writing, software  
 10   +// distributed under the License is distributed on an "AS IS" BASIS,  
 11   +// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
 12   +// See the License for the specific language governing permissions and  
 13   +// limitations under the License.  
 14   +  
 15   +  
 16   +#pragma once  
 17   +  
 18   +#define IDS_PROJNAME                    100  
 19   +#define IDR_PERSISTENTOPTOUT            101  
 20   +#define IDR_PERSISTENTOPTOUTBHO         102  
 


 
0 comments on commit 65c5678 
  1   +With this browser plugin you can permanently opt out of the DoubleClick cookie, which is an advertising cookie that Google uses.  
 2   +The plugin lets you keep your opt-out status for this browser even when you clear all cookies.  
 3   +  
 4   +The project contains both Javascript code for a Firefox extension, and C++ code for an IE Add-on (dll).  
