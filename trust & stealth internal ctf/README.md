trust와 stealth선배들이 수학여행을 갔다왔을때 열린 ctf
===================================================
  
# 췍췍 MC조정훈입니다. - 10000
'''
SHANGHAI{잘다녀올게얘들아}
'''


# Encode our life - 75
>Ascii85로 ;b02L7n>p;HVR@9?T'_n/hT15I/을 디코딩해준다.

'''
SHANGHAI{NOT_64...:(}
'''


# TOP SECRET - 125
>pptx를 받아서 열어보면  
>QRcode가 두개로 찢겨져서 숨겨져있다.  
>둘을 적당히 조합해서 코드를 찍으면?  

###사이트로 이동 (https://m.imgur.com/a/~~~~~)
>하얀배경을 확대시켜보면 안에 플래그가있다.  
'''
SHANGHAI{ripped_qrcode_:(}
'''

# TRUSEALTH - 150
>TRUST는 0으로 STEALTH는 1로 바꿔서 문자열로 변환시키면 플래그가 나온다.  
'''
SHANGHAI{string_to_binary_to_trustealth}
'''  
# zip1337
>압축파일을 1337번 해제하면 플래그가 나온다.  
>딱봐도 매크로 짜는 문제인거같아서 파이썬으로 코드를 만들었다.  
>안전하게 2에서 멈추고 2번정도는 수동으로 했다.

'''
import zipfile


zip = zipfile.ZipFile('C:\\~~~\\flag1337.zip')
zip.extractall('C:\\~~~\\flag1337')
c=1336
while True:
    a = 'C:\\~~~\\flag' + str(c) + '.zip'
    print(a)
    zip = zipfile.ZipFile(a)
    zip.extractall('C:~~~\\~~~\\flag1337')
    c = c - 1
    if c == 2:
        break

zip.close()
'''
