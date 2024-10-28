# -*- coding: 1252 -*-
### -*- coding: iso-8859-1 -*-




from sight import *
from sieve import *
from functools import partial
import PIL
socket.setdefaulttimeout(1)

import concurrent.futures

def concurrent(fc, li):
    with concurrent.futures.ThreadPoolExecutor() as executor:
        futuros = [executor.submit(fc, et) for et in li]
        resultados = [f.result() for f in futuros]
    return resultados



dicionarios=\
'''
tree:
    s.dtx[ti]=dr         
    s.dti[ti]=item
    s.dxt[dr]=ti

tred:     
    s.dtf[ti]=fn
    s.dtd[ti]=item
    s.dft[fn]=ti
    
    s.dtk[ti]=ky
    s.dkt[ky]=ti

frame:
    s.dfc[fn]=cv
    s.dcf[cv]=fn

keysValues:
    s.dkv
'''

def dicio():
    print(dicionarios)


def fitHeight(cw,ch,iw,ih):

    iw=iw*ch/ih
    ih=ch    
    return int(iw),int(ih)

def fitWidth(cw,ch,iw,ih):

    ih=ih*cw/iw
    iw=cw
    return int(iw),int(ih)

class AutoScrollbar(Scrollbar):
   
    def set(s,lo,hi):
        
        if float(lo)<=0.0 and float(hi)>=1.0:           
            s.tk.call('grid','remove',s)
        else:
            s.grid()
        Scrollbar.set(s,lo,hi)

def files(dr):    
    
    for ps,ds,fs in os.walk(dr):
        for bn in fs:            
            fl=pathjoin(ps,bn)
            yield fl

def folderLen(dr):

    li=tuple(files(dr))
    return len(li)

def folderSize(dr):
    
    fs=files(dr)
    return sum([getsize(fl) for fl in fs])
               
def getMtime(aq):

    mt=os.path.getmtime(aq)
    lt=localtime(mt)[:6]    
    st='%.4d/%.2d/%.2d %.2d:%.2d:%.2d'%lt
    return st

def invDat(dt):
    
    dat=str(dt)

    try:
        x,m,y,H,M,S=re.split('/| |:',dat)
        return '%s/%s/%s %s:%s:%s'%(y,m,x,H,M,S)
    except:
        return dt

def icSort(ls,i=0):

    ax=[(et[i],et) for et in ls]
    ax.sort()
    return [et for (ic,et) in ax]

def fmSize(sz):   
    
    m=1024.0    

    if sz<m:
        vl='%.2d Bt'%sz

    elif m<sz<m**2:
        rs=float(sz/m)
        vl='%.2f Kb'%rs

    elif m**2<sz<m**3:
        rs=float(sz/m**2)
        vl='%.2f Mb'%rs
        
    elif sz>m**3:
        rs=float(sz/m**3)
        vl='%.2f Gb'%rs

    else:
        vl=None      
        
    return vl

def fz2sz(fz):

    m=1024.0

    sp=fz.split()
    sz,tp=sp
    if tp=='Bt':
        sz=int(sz)
    elif tp=='Kb':
        sz=int(sz)*m
    elif tp=='Mb':
        sz=int(sz)*m*m
    elif tp=='Gb':
        sz=int(sz)*m*m*m
    return sz

def listfs(path):

    ld=os.listdir(path)

    for et in ld:
        fl=pjoin(path,et)
        if isfile(fl):
            yield(fl)        

def listds(path):

    ld=os.listdir(path)

    for et in ld:
        dr=pjoin(path,et)
        if isdir(dr):
            yield(dr)

def isImage(fl):
    
    try:
        im=Image.open(fl)
        w,h=im.size
        del im
        return True
    except:
        return False

if not isdir('temp'):
    os.mkdir('temp')

if not isdir('trash'):
    os.mkdir('trash')

def clearTemp():
    tempFiles=folderFiles('temp')
    try:
        for fl in tempFiles:
            os.remove(fl)
    except Exception as ex:
        print(fl)
        print(ex)
        print()
    
    tempDirs=listds('temp')
    try:
        for dr in tempDirs:
            shutil.rmtree(dr)
    except Exception as ex:
        print(dr)
        print(ex)
        print()

def clearTrash():

    trashFiles=folderFiles('trash')
    for fl in trashFiles:
        os.remove(fl)

    trashDirs=listds('trash')
    for dr in trashDirs:
        shutil.rmtree(dr)



def getExts(tl):
   
    li=[]
    
    for ln in tl:
        try:
            nm,ex=splitext(ln)
            li.append(ex.lower()[:4].replace('?',''))
        except:
            pass
        
    dc={}        
    for e in li:
        le=len(e)
        if e in dc:
            dc[e]+=[e]
        else:
            dc[e]=[e]
                
    di=dc.items()        
    di=[(k,len(v)) for (k,v) in di]
           
    d2=[(v,k) for (k,v) in di]        
         
    d2.sort()
    d2=d2[::-1]
    d3=[('%d'%k,v) for (k,v) in d2]
    d4=[k+v.rstrip(punctuation) for k,v in d3]               
    tx='  '.join(d4)
   
    return tx

def sortUris(urs):

    try:
        rs=[]
        nu=[]
        for ui in urs:
            bn=basename(ui)        
            ns=[]            
            for e in bn:
                if e in list('1234567890'):
                    ns.append(e)
            if len(ns)>0:
                nr=int(''.join(ns))      
                nu.append((nr,ui))
            else:
                pass
           
        nu.sort()
        
        for n,u in nu:
            rs.append(u)
        return rs
    
    except:
        return urs

def groupDirs(li):    

    li=[et for et in li if et!='']
    dc={}
    pt=set()
    nr=0
    for et in li:
        dn=dirname(et)
        pt.add(et)
        nx=near(li,et)       
        dx=dirname(nx)        
        if dn==dx:            
            pt.add(nx)
        else:
            nr+=1
            ky='%s-%d'%(dn,nr)
            dc[ky]=tuple(pt)
            pt=set()            
    return dc

def near(li,el,dt=1):
    
    ic=li.index(el)+dt
    ic=ic%len(li)
    return li[ic]

def statusGet():

    ext=splitext(s.path)[1]
    
    if isdir(s.path):
        ffs=folderFiles(s.path)
        return len(ffs)
        
    elif isfile(s.path):
        if ext in dkvExt:
            tis=s.tred.get_children()            
            return len(tis)
        else: 
            par=dirname(s.path)
            ffs=folderFiles(par)
            return len(filtrEnd(ffs,[ext]))

def printex(wt,ex):
    print(wt)
    print(ex)
    print()

def zipQuant(zp):
    # Tamanho mínimo do EOCD é 22 bytes (sem o comentário)
    eocd_size = 22
    signature = b'\x50\x4b\x05\x06'  # Assinatura EOCD

    with open(zp, 'rb') as f:
        # Vai para o final do arquivo para buscar o EOCD
        f.seek(-eocd_size, 2)
        eocd_data = f.read()

        # Verifica se encontramos a assinatura do EOCD
        if signature in eocd_data:
            # Offset da assinatura até o início do EOCD
            eocd_offset = eocd_data.index(signature)
            
            # Move o ponteiro para o início do EOCD
            f.seek(-eocd_size + eocd_offset, 2)
            eocd_data = f.read(eocd_size)
            
            # Extrai o número total de entradas no diretório central
            total_entries = int.from_bytes(eocd_data[10:12], 'little')
            return total_entries
        else:
            raise ValueError("EOCD não encontrado no arquivo ZIP")

class Mtreeview:

    def __init__(s,ms):

        s.ms=ms
        tit=basename(sys.argv[0])
        s.ms.title('ttkFolderSizes - %s'%tit)

        s.ms.protocol('WM_DELETE_WINDOW',Fc(s.sair))
                
        #Start Menus

        s.Men=Menu(s.ms,tearoff=0)
        s.ms.config(menu=s.Men)        

        men=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Arquivo',menu=men)

        la=['Abrir diretório',
            'Colar end diretório',
            'Abrir arquivo',
            'Colar end arquivo',
            'Ver no explorer',
            'Ver no browser',
            'Remover do disco',
            'Salvar alterações',
            'Lançar arquivo',
            ]
        
        fa=[s.abrirDiretorio,
            s.colarEndDiretorio,
            s.abrirArquivo,
            s.colarEndArquivo,
            s.verNoExplorer,
            s.verNoBrowser,
            s.removerDoDisco,
            s.salvarAlteracoes,
            s.lancarArquivo,
            ]       
        
        za=list(zip(la,fa))
        za.sort()
        
        for l,f in za:            
            men.add_command(label=l,command=f)

        mnf=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Filtro',menu=mnf)

        lf=['All','Any','End','Start','None']
        lf.sort()

        for ft in lf:
            mnf.add_command(label=ft,command=Fc(s.setFilt,ft))

        mne=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Editar',menu=mne)

        le=['Copiar texto',
            'Colar texto',
            'Copiar para',
            'Remover do disco',
            'Lançar arquivo',
            'Selecionar tudo',
            'Selecionar nada',
            'Renomear',
            ]
        
        fe=[s.copiarTexto,
            s.proExpand,
            s.copiarPara,
            s.removerDoDisco,
            s.lancarArquivo,
            s.selecionarTudo,
            s.selecionarNada,
            s.renomear,
            ]

        ze=list(zip(le,fe))
        ze.sort()
        
        for lb,fc in ze:
            mne.add_command(label=lb,command=fc)

        mnt=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Tools',menu=mnt)
        
        lt=[            
            'Userprofile',            
            'Tamanho do lado',
            'Recursivo',          
            'Scan All Drivers',
            'Zippar',
            'Ver arquivos temporários',
            'Limpar lixeiras',
            'Reset files hist',
             ]
        
        ft=[            
            s.userprofile,            
            s.tamanhoDoLado,
            s.recursivo,
            s.scanAllDrivers,
            s.zippar,
            s.verArquivosTemporarios,
            s.limparTemps,
            s.resetFilesHist,
            ]
        
        zt=list(zip(lt,ft))
        zt.sort()
        for lb,fc in zt:
            mnt.add_command(label=lb,command=fc)        
        
        msz=Menu(s.Men,tearoff=0)

        lz=['Width','Height','None']

        for lb in lz:
            msz.add_command(label=lb,command=s.setResize(lb))
            
        mns=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Selecionado',menu=mns)

        ls=[
            'Copiar texto',
            'Remover do disco',            
            'Produzir HTML',
            'Selecionar tudo',
            'Selecionar nada',
            'Copiar para',
            'Mover para',
            'Zippar',
            'TextName2Web',
            'Lancar arquivo',
            'Expandir linha',
            'Text find',
            'Url2links',
            ]
        
        fs=[s.copiarTexto,
            s.removerDoDisco,            
            s.produzirHtml,
            s.selecionarTudo,
            s.selecionarNada,
            s.copiarPara,
            s.moverPara,
            s.zippar,
            s.textName2Web,
            s.lancarArquivo,
            s.expandirLinha,
            s.findInTex,
            s.url2links,
            ]

        zs=list(zip(ls,fs))
        zs.sort()

        for (lb,cm) in zs:
            mns.add_command(label=lb,command=cm)        

        s.mnm=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Imagem',menu=s.mnm)

        lm=['Flip horizontal',
            'Flip vertical',
            'Rotacionar imagem',
            'Copiar para',
            'Remover do disco',
            'Browse Max Dims',
            'Recompor arquivo',
            ]
        
        lf=[s.flipHorizontal,
            s.flipVertical,
            s.rotacionarImagem,
            s.copiarPara,
            s.removerDoDisco,
            s.browseMaxDims,
            s.recomporArquivo,
            ]

        zm=list(zip(lm,lf))
        zm.sort()
        
        for lb,fc in zm:
            s.mnm.add_command(label=lb,command=fc)

        ind=listind(lm,'remover')        
        s.mnm.insert_separator(ind+1)
        s.mnm.insert_separator(ind+3)

        mnd=Menu(tearoff=0)
        s.Men.add_cascade(label='Drivers',menu=mnd)

        drvs=list('defghijklmnoprstuvwxyz'.upper())
        for ic,nm in enumerate(drvs):
            if isdir('%s:/'%drvs[ic]):                
                mnd.add_command(label='%s:/'%nm,command=Fc(s.scanDir,nm))

        
        
        msr=Menu(s.mnm,tearoff=0)
        s.mnm.add_cascade(label='Resize',menu=msr)
        
        lz=['Width','Height','None']

        for lb in lz:
            msr.add_command(label=lb,command=s.setResize(lb))

        mod=Menu(s.mnm,tearoff=0)
        s.mnm.add_cascade(label='Ordenar figuras',menu=mod)

        ods=['Original','Alfa crecente','Alfa decrescente','aleatória']

        for od in ods:
            mod.add_command(label=od,command=Fc(s.setOrdFigs,od))             

        s.mnw=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Texto',menu=s.mnw)
        
        mnt=Menu(s.mnw,tearoff=0)
        s.mnw.add_cascade(label='Wrap',menu=mnt)        

        l2=[
            'none',
            'word',
            'char',
            ]
        
        l2.sort()

        for lb in l2:
            mnt.add_command(label=lb.capitalize(),command=Fc(s.setWrap,lb))

        mnc=Menu(s.mnw,tearoff=0)
        s.mnw.add_cascade(label='Encoding',menu=mnc)        

        cods=['get enc','utf-8-sig','ANSI','utf-8','iso-8859-1',
              'cp1252','latin1','UTF-8 com BOM','UTF-16 LE']
        
        cods.sort()

        for lb in cods:
            mnc.add_command(label=lb.capitalize(),command=Fc(s.setEncoding,lb))

        lt=[('Copiar',s.copiarTexto),
            ('Single lines',s.singleLines),
            ('Salvar texto como',s.salvarTextoComo),
            ('Salvar alterações',s.salvarAlteracoes),
            ('Unquote',s.unquoteText),
            ('Group dirs',s.groupDirs),
            ('Text 2 dkv',s.text2dkv),
            ('UrVs 2 dkv',s.uvs2dkv),
            ('Salvar dkv',s.salvarDkv),
            ('Mostrar dkv maiores',s.mostrarDkvsMaiores),
            ('Text 2 kvExt',s.text2kvExt),
            ('Prettify',s.prettify),
            ('Names to Urls',s.names2urls),
            ]

        for lb,cm in lt:
            s.mnw.add_command(label=lb, command=cm)

        mex=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='URL links',menu=mex)

        mf=[('Span figs',s.spanFigs),
            ('Colar',s.proExpand),
            ('splitLinks',splitLinks),
            ('reqLinks'  ,reqLinks),
            ('fig2Links' ,fig2Links),            
            ('specLinks' ,specLinks),
            ('figLS'     ,figLS),
            ('getLinks'  ,getLinks),           
            ('splitSources',splitSources),
            ('soupSources',soupSources),
            ('dobFig2Links',dobFig2Links),
            ('concUrlFigs',concUrlFigs),
            ]

        mts,fcs=list(zip(*mf))
       
        mts=[e.strip() for e in mts]
        mts.sort()

        mex.add_command(label='Colar',command=s.proExpand)
        for mt in mts[1:]:
            mex.add_command(label=mt,command=Fc(s.setMet,mt))

        mex.add_command(label='Zippar',command=s.zippar)

        mno=Menu(s.mnw,tearoff=0)
        s.mnw.add_cascade(label='Ordem',menu=mno)

        lo=['Inversa','Original','Alfabética','Aleatória']

        for lb in lo:
            mno.add_command(label=lb,command=Fc(s.setTextOrd,lb))         

        mnh=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Histórico',menu=mnh)

        lh=['Diretórios',
            'Arquivos',
            'Figuras',
            'Favoritos',
            'Expandidas',            
            'Urls',
            ]
        
        fh=[s.verDirsHist,
            s.verFilesHist,
            s.verFigsHist,
            s.verFavoritos,
            s.verExpandHist,            
            s.verUrlsHist,
            ]

        zh=list(zip(lh,fh))
        zh.sort()
        for lb,fc in zh:
            mnh.add_command(label=lb,command=fc)

        mot=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='Outros',menu=mot)

        lo=['Quantidade de figuras',
            'Show mask',
            'Favoritos',
            'Toggle entry',
            'Shuffle columns',
            'Tela cheia',
            'Tela padrão',
            'Span canvas',
            'Zipar dkv',
            'MisturarColunas',
            'Salvar todas as figuras',
            'Acrescentar diretorio aos favoritos',
            ]

        fo=[s.qtdFigs,
            s.showMask,
            s.verFavoritos,
            s.togEntry,
            s.misturarColunas,
            s.telaCheia,
            s.telaPadrao,
            s.spanCanvas,
            s.zipparDkv,
            s.misturarColunas,
            s.salvarTodasAsFiguras,
            s.acrescentarDiretorioAosFavoritos,
            ]

        zo=list(zip(lo,fo))
        zo.sort()

        for ot,fc in zo:
            mot.add_cascade(label=ot,command=fc)

        kvm=Menu(s.Men,tearoff=0)
        s.Men.add_cascade(label='DKVS',menu=kvm)

        lk=['Mais de cinco ítens',
            'Misturar colunas',
            'Zippar dkv',
            'Salvar dkv',
            'Remover repetidos no dkv',
            ]
        
        fk=[s.dkvs5mais,
            s.misturarColunas,
            s.zipparDkv,
            s.salvarDkv,
            s.removerRepetidosNoDkv,
            ]
        
        zk=list(zip(lk,fk))
        zk.sort()
        for lb,fc in zk:
            kvm.add_command(label=lb,command=fc) 
        
        nfo=Menu(s.Men,tearoff=0)
        s.Men.add_command(label='Info',command=s.info)

        #End Menus

        #Start gui

        s.frs=Frame(s.ms)
        s.frs.grid(row=0,column=1,sticky=W+E)
        s.frs.columnconfigure(0,weight=1)

        s.tit=Label(s.frs,fg='blue')
        s.tit.grid(row=0,column=0,ipadx=10,sticky=W+E)
        
        s.frd=Frame(s.frs)
        s.frd.grid(row=0,column=1,padx=2)

        s.sve=StringVar()
        s.wEf=Pmw.EntryField(s.frs,
                             labelpos=W,
                             entry_foreground='blue',
                             entry_font=font1,
                             entry_textvariable=s.sve,
                             )
        s.wEf.grid(row=1,column=0,sticky=W+E,columnspan=2)

        s.lab=s.wEf.component('label')
        s.ent=s.wEf.component('entry')

        s.lab.config(text='Clear',width=6,fg='red')
        s.lab.bind('<1>',Fc(s.clearEntry))
        
        s.fdr=Frame(s.ms)        
        s.fdr.grid(row=1,column=2,sticky=W)        

        #buttons

        gs=[
            'm_catprio.gif',
            'and.gif',
            'display 16x16.gif',
            
            'remove.gif',
            't_next.gif',
            'stats_7.gif', 

            'down-1.gif',
            'radiation.gif',
            'art.gif',
            
            'coqueiro.gif',            
            '3d bar chart.gif',
            'text.gif',
            
            'report.gif',            
            'windows.gif',
            'world.gif',            
            ]

        
        cm=[
            Fc(s.misturarColunas),
            Fc(s.udcFigs),
            Fc(s.thumbnails),

            Fc(s.showFigs,-1),            
            Fc(s.showFigs,0),            
            Fc(s.showFigs,1),            
            
            Fc(s.refreshFolder),            
            Fc(s.lancarArquivo),            
            Fc(s.liftWidget,'canvas'),
            
            Fc(s.liftWidget,'tree'),             
            Fc(s.liftWidget,'tred'),            
            Fc(s.liftWidget,'text'),
            
            Fc(s.liftWidget,'box'),            
            Fc(s.liftWidget,'frame'),            
            Fc(s.produzirHtml), 
            ]

        os=[
            'Misturar colunas',
            'Udc figs',
            'Ini & End => Thumbs',

            'Miniaturas anteriores',            
            'Refresh atuais',            
            'Miniaturas posteriores',            
            
            'Refresh folder',
            'Lançar arquivo',            
            'Lift canvas',
            
            'Lift tree',            
            'Lift tred',            
            'Lift text',
            
            'Lift box',            
            'Lift frame',            
            'Produzir Html',             
            ]

        zp=zip(gs,cm,os)
        
        bs=[]
        for ic,(gf,cm,oo) in enumerate(zp):
            im=Image.open('gifs/%s'%gf)
            im=im.resize((16,16))
            s.ph=ImageTk.PhotoImage(im)
            
            bt=Button(s.fdr,                      
                      image=s.ph,
                      width=30,
                      relief="flat",
                      overrelief="raised",
                      height=16,
                      command=cm,)
            bt.image=s.ph
            
            bt.grid(row=ic,column=0,sticky=W,pady=1)
            bs.append(bt)
            bl=Pmw.Balloon(xoffset=-100,yoffset=-25)
            bl.bind(bt,oo)        

        s.ftr=Frame(s.ms)
        grid1(s.ftr,col=1)
        redim(s.ftr)

        s.columns=(
                   u'Arquivos',
                   u'Ext',
                   u'Itens',
                   u'Tamanho',
                   u'Data',
                   u'Observação',
                   )

        anchors=(W,CENTER,CENTER,CENTER,CENTER,W)
        
        s.tree=ttk.Treeview(s.ftr,
                            columns=s.columns,
                            show='headings',
                            padding=(5,0,5,0),
                            selectmode='extended',
                            )
        grid1(s.tree)

        zca=zip(s.columns,anchors)
  
        for col,anc in zca:
            s.tree.heading(col,
                           text=col,
                           anchor=anc,
                           command=Fc(s.sortColumn,col),
                           )

        vsb=AutoScrollbar(s.ftr,orient='vertical', command=s.tree.yview)        
        s.tree.configure(yscrollcommand=vsb.set)
        vsb.grid(row=1,column=2,sticky=N+S)

        hsb=AutoScrollbar(s.ftr,orient='horizontal', command=s.tree.xview)        
        s.tree.configure(xscrollcommand=hsb.set)
        hsb.grid(row=2,column=1,sticky=W+E)        

        cfs=[             
             ('#1',  300, W),
             ('#2',   50, E),
             ('#3',   60, E),
             ('#4',   80, E),
             ('#5',  120, CENTER),
             ('#6',  300, W),
             ]
        
        for nr,wd,ac in cfs:
            t=s.tree.column(nr,
                            width=wd,
                            anchor=ac,
                            stretch=NO,
                            )

        sty=ttk.Style()        
        sty.configure("Treeview",
                      foreground='black',                      
                      )
       
        s.frTd=Frame(s.ms)
        grid1(s.frTd)
        redim(s.frTd)        
        
        s.tred=ttk.Treeview(s.frTd,
                            columns=[],
                            show='headings',
                            padding=(5,0,5,0),
                            selectmode='extended',
                            )
        grid1(s.tred)  

        vsb=AutoScrollbar(s.frTd,orient='vertical', command=s.tred.yview)        
        s.tred.configure(yscrollcommand=vsb.set)
        vsb.grid(row=1,column=2,sticky=N+S)

        hsb=AutoScrollbar(s.frTd,orient='horizontal', command=s.tred.xview)        
        s.tred.configure(xscrollcommand=hsb.set)
        hsb.grid(row=2,column=1,sticky=W+E)        

        s.frb=Frame(s.ms,borderwidth=0)        
        s.frb.grid(row=0,column=1) 
        
        s.wCv=Pmw.ScrolledCanvas(s.ms,                                 
                                 canvas_highlightthicknes=0,
                                 canvas_borderwidth=0,
                                 canvas_confine=True,
                                 canvas_background='goldenrod2',
                                 )
        grid1(s.wCv)
        s.canvas=s.wCv.interior()

        s.wBx=Pmw.ScrolledListBox(s.ms,
                                  listbox_foreground='blue',
                                  listbox_font=font1,
                                  listbox_selectmode='extended',
                                  )
        grid1(s.wBx)        
        s.box=s.wBx.component('listbox')

        s.wTx=Pmw.ScrolledText(s.ms,
                               text_foreground='purple',
                               text_background='White',
                               text_font=font1,
                               text_wrap=NONE,
                               text_undo=True,
                               )
        grid1(s.wTx)
        s.tex=s.wTx.component('text')

        s.wFr=Pmw.ScrolledFrame(s.ms,
                                frame_background='SystemButtonFace',
                                frame_highlightthicknes=0,
                                )
        grid1(s.wFr)
        
        s.frame=s.wFr.component('frame')

        s.fri=Frame(s.ms)
        s.ms.columnconfigure(1,weight=1)
        s.fri.grid(row=3,column=1,sticky=E)

        s.sta=Label(s.fri,fg='blue',justify=RIGHT)
        s.sta.grid(row=0,column=0,sticky=E)

        s.stb=Label(s.fri,relief='sunken',fg='red')
        s.stb.grid(row=0,column=1,sticky=W+E,padx=5,ipadx=5)

        s.stc=Label(s.fri,fg='blue')
        s.stc.grid(row=0,column=2,padx=5)

        s.std=Label(s.fri,fg='red')
        s.std.grid(row=0,column=3,padx=5)

        #End gui
        #binds

        s.tit.bind('<1>',Fc(s.seeParent))

        s.ent.bind('<Up>'      ,Fc(s.rollUrl,1))
        s.ent.bind('<Down>'    ,Fc(s.rollUrl,-1))
        s.ent.bind('<Return>'  ,s.fromEntry)

        s.ent.bind('<Insert>',Fc(s.executar))
        s.ent.bind('<F12>',Fc(s.findInFiles))
        
        s.ms.bind('<Prior>',Fc(s.scrollPage,-1))
        s.ms.bind('<Next>',Fc(s.scrollPage,1))        
        s.ms.bind('<MouseWheel>',s.mouseScroll)
        s.ms.bind('<BackSpace>',Fc(s.nearFigs,-1))
        s.ms.bind('<space>',Fc(s.nearFigs,1))

        s.ms.bind('<Control-Home>',Fc(s.gotoHome))
        s.ms.bind('<Control-End>',Fc(s.gotoEnd))        

        s.ms.bind('<3>',s.pop3)
        s.ms.bind('<Delete>',Fc(s.removerDoDisco))
        
        s.ms.bind('<Control-c>',Fc(s.copiarTexto))
        s.ms.bind('<Control-C>',Fc(s.copiarTexto))
        s.ms.bind('<Control-s>',Fc(s.salvarAlteracoes))
        s.ms.bind('<Control-S>',Fc(s.salvarAlteracoes))
        s.ms.bind('<Control-v>',Fc(s.proExpand))
        s.ms.bind('<Control-V>',Fc(s.proExpand))

        s.ms.bind('<Control-t>',Fc(s.iniSlider))
        s.ms.bind('<Control-y>',Fc(s.endSlider))
        
        s.ms.bind('<Alt-z>',Fc(s.seeParent))
        s.ms.bind('<Alt-Z>',Fc(s.seeParent))
        s.ms.bind('<Alt-x>',Fc(s.togEntry))
        s.ms.bind('<Alt-X>',Fc(s.togEntry))
        
        s.ms.bind('<Control-a>',Fc(s.selecionarTudo))
        s.ms.bind('<Control-A>',Fc(s.selecionarTudo))
        s.ms.bind('<Alt-a>',Fc(s.selecionarNada))
        s.ms.bind('<Alt-A>',Fc(s.selecionarNada))
        
        s.ms.bind('<F1>',Fc(s.verTreeFocus))
        s.ms.bind('<F2>',Fc(s.clearEntry))

        s.ms.bind('<F3>',Fc(s.togEntry))
        s.ms.bind('<F4>',Fc(s.refreshFolder))
        s.ms.bind('<F5>',Fc(s.repriseData))
        s.ms.bind('<F7>',Fc(s.selecionarNada))
        s.ms.bind('<F8>',Fc(s.telaInicial))
        s.ms.bind('<F10>',Fc(s.telaPadrao))
        s.ms.bind('<F11>',Fc(s.togleScreen))

        s.ms.bind('<Escape>',Fc(s.togWidget))
        s.ms.bind('<Pause>',Fc(s.parar))

        s.ms.bind('<Left>',Fc(s.nearFiles,-1))        
        s.ms.bind('<Right>',Fc(s.nearFiles,1))
        
        s.ms.unbind_all("<<NextWindow>>")
        s.ms.bind('<Tab>',Fc(s.changeResize))

        s.tree.bind('<Double-1>',Fc(s.fromTree))
        s.tree.bind('<Return>',Fc(s.fromTree))
        s.tree.bind('<KeyPress>',s.keyPress)        
        s.tree.bind('<<TreeviewSelect>>',s.focusLine)
        
        s.tred.bind('<<TreeviewSelect>>',s.focusLine)
        s.tred.bind('<Double-1>',Fc(s.fromTred))
        s.tred.bind('<Return>',Fc(s.fromTred))
        s.tred.bind('<KeyPress>',s.keyPress)

        s.tex.bind('<Double-1>',Fc(s.fromText))
        s.tex.bind('<Control-Shift-Key-Z>',s.redoText)
        s.tex.bind('<1>',Fc(s.showTexIndic))        
        
        s.canvas.bind('<1>',Fc(s.liftAnterior))
        s.canvas.bind('<Return>',Fc(s.nearFigs,1))
        

        s.sta.bind('<1>',Fc(s.copiarPath))
        s.stb.bind('<1>',Fc(s.showFigs,1))
        
        s.box.bind('<Double-1>',Fc(s.fromBox))
        s.stc.bind('<1>',Fc(s.liftAnterior))
        s.widgets=['tree','frame','tred','canvas','text','box']


        ##########################################################
        

        s.telaPadrao()
               
        s.initials()
        s.lembrarConfigs()      
        s.criarRenomeador()
        s.criarInfoText()
        s.liftWidget('tree')

        s.ms.update()
        s.wEf.grid_remove()

        s.telaInicial()
        s.criarScTexTopLev()
        s.ms.title('ttk Folder Sizes')
        

##        clearTemp()
##        clearTrash()
        

        ##########################################################
        

    def changeFgRed(s,ev):

        wg=ev.widget        
        wg['fg']='red'

    def changeFgBlue(s,ev):
        wg=ev.widget        
        wg['fg']='blue'       


    def expandirLinha(s):

        if all([s.lift=='tred',s.tredCols==['Nr','Urls','Qtds']]):
            ti=s.tred.focus()
            fp=s.dtk[ti]
            vs=s.dkv[fp]
            s.ky=fp

        elif s.lift=='tree':
            ti=s.tree.focus()
            fp=s.dtx[ti]            
            
            if isfile(fp):
                
                if fp.endswith(tuple(textExt)):                
                    vs=textLines(fp)
                    
                elif fp.endswith(tuple(zipExt)):
                    zf=ZipFile(fp)
                    zn=zf.namelist()
                    vs=[nm2ur(fn) for fn in zn]
                    
                elif fp.endswith(tuple(dkvExt)):
                    kv=openDkv(fp)
                    vls=list(kv.values())
                    vs=[]
                    for et in vls:
                        try:
                            vl=et[0]
                            vs.append(vl)
                        except:
                            pass                    

        s.wBx.setlist(vs)
        s.liftWidget('box')  

        tt=len(vs)
        s.ant='box'
        
        s.wTx.configure(text_foreground='purple')
        upath=unquote(s.path)
        s.tit.config(text=upath)
        
        tx=basename(fp)[-172:]
        tx=unquote(tx)
        
        s.sta.config(text=tx)
        s.stb.config(text='%d lines'%tt)
        s.stc.config(text='lift: %s enc: %s'%(s.lift,s.enc))

        s.figs=filtrAny(vs)

    def setTextOrd(s,od):

        if od=='Original':
            s.figs=filtrAny(s.textlines)
            s.wTx.settext('\n'.join(s.textlines))
            
        elif od=='Inversa':
            ga=s.tex.get('0.0',END)
            ax=ga.split('\n')
            inv=ax[::-1]
            s.figs=filtrAny(inv)
            s.wTx.settext('\n'.join(inv))
            
        elif od=='Alfabética':
            cp=s.textlines[:]
            cp.sort()
            s.figs=filtrAny(cp)
            s.wTx.settext('\n'.join(cp))

        elif od=='Aleatória':
            cp=s.textlines[:]
            shuffle(cp)
            s.figs=filtrAny(cp)
            s.wTx.settext('\n'.join(cp))        

    def redoText(s,ev):

        s.tex.event_generate('<Control-y>')

    def telaPadrao(s):

        s.ms.state('normal')
        s.ms.geometry('930x450+100+40')

    def focusLine(s,ev):
        
        try:
            if s.lift=='tree':
                ti=s.tree.focus()
                gt=s.dtx[ti]
                s.tree.see(ti)
                
            elif s.lift=='tred':
                ti=s.tred.focus()
                gt=s.dtk[ti]
                s.tred.see(ti)
                
            s.sta.config(text=unquote(gt[-72:]))
                
            if all([gt.endswith(tuple(imagExt)),isfile(gt)]):
                try:
                    im=Image.open(gt)
                    wd,he=im.size
                    s.stb.config(text='%d x %d'%(wd,he))
                except Exception as ex:
                    printex('focusLine',ex)
                    
        except:
            pass

    def path2clipboard(s):
       
        path=s.path  
        s.ms.clipboard_clear()
        s.ms.clipboard_append(path)

        tp=Toplevel()
        lb=Label(tp,
                 text='Copiado:\n%s\n'%path,
                 fg='red',
                 font=('Times',12),
                 )
        lb.grid(padx=20, pady=20)
        tp.update()
        centralizar(tp)
        tp.after(4000,Fc(tp.destroy))

    def sobre(s):

        prog=sys.argv[0]        
        bn=basename(prog)
        showinfo('Sobre',bn)

    def togEntry(s):

        if s.wEf.winfo_ismapped():
            s.entryOut()
            
        else:
            s.entryIn()
            

    def entryIn(s):

        s.wEf.grid()
        s.ms.unbind('<Tab>')
        s.ms.unbind('<space>')
        s.ms.unbind('<BackSpace>')
        s.ms.unbind('<Delete>')
        s.ms.unbind('<Right>')
        s.ms.unbind('<Left>')
        s.ent.focus_force()

    def entryOut(s):

        s.wEf.grid_remove()
        s.ms.bind('<Tab>',Fc(s.changeResize))
        s.ms.bind('<space>',Fc(s.nearFigs,1))
        s.ms.bind('<BackSpace>',Fc(s.nearFigs,-1))
        s.ms.bind('<Delete>',Fc(s.removerDoDisco))
        s.ms.bind('<Right>',Fc(s.nearFiles,1))
        s.ms.bind('<Left>',Fc(s.nearFiles,-1))           
        s.tit.focus_force()

    def initials(s):

        s.tid=None

        s.dbs=dict()

        s.url=''
        s.scrFig=''
        s.scrRef=0
        s.reFis=[]
        s.fks=[]
        s.zpNs=[]
        s.acts=[]

        s.met='getLinks'

        s.zipUdcRef=[]
        
        s.zip=''

        s.fis=[]
        s.fgKv=dict()
        s.fgZp=dict()
        s.fgPt=dict()
        s.urlsHist=dict()
        s.dirsHist=dict()
        s.expandHist=dict()

        s.tmFg=list()

        s.dkv=dict()
        s.dvk=dict()       

        s.key=None
        s.keys=[]
        
        s.favoritos=[
                     'D:/pyDevelop3',
                     'D:/pyDevelop3/dataReader',
                     'D:/pyDevelop3/webSpy/DKVs',
                     'D:/pyDevelop3/webSpy/textos',
                     'D:/pyDevelop3/webSpy/keysValues',                     
                     ]
        
        s.lift=''
        s.enc='utf-8'
        s.path=''

        s.stop=0
        s.ini=0
        s.end=50
        s.qtd=50
        
        s.filt='Any'
        s.ant='tree'

        s.rev=1
        s.path=sys.argv[0]
        s.figs=[]
        s.orig=s.figs[:]
        s.files=[]
        
        s.data=tuple([])
        s.filesHist=dict()
        s.tredCols=[]
        s.drVals=[]
        s.flVals=[]
        s.fis=[]
        s.fks=[]
        
        s.datas=dict()

        s.dnt=dict()
        s.dtf=dict()
        
        s.dxt=dict()
        s.dtx=dict()
        s.dti=dict()
        
        s.dkv=dict()
        s.dfc=dict()
        s.dcf=dict()

        s.dtk=dict()
        s.dtd=dict()
        
        s.rzf=None
        s.rzl=[]
        s.rzn=[]
        
        s.rsz='Height'
        s.fig=''
        s.figs=[]
        s.orig=s.figs[:]        
        
        s.ldo=180
        
        s.textlines=[]
        s.clip=None
        s.dst=''
        s.read=''   
        
    def pop3(s,ev):

        s.Men.post(ev.x_root,ev.y_root)

    def abrirDiretorio(s):

        ad=askdirectory(initialdir='')
        s.ms.update()

        if ad:
            s.delTree()
            s.liftWidget('tree')
            s.lift='tree'
            s.ant='tree'
            s.path=ad
            s.salvarConfigs()
            s.scanPath()

    def colarEndDiretorio(s):

        s.delTree()
        s.initials()
        cg=s.ms.clipboard_get()
        cg=cg.replace('file:///','')
        
        if isdir(cg):
            s.path=cg
            s.salvarConfigs()
            s.scanPath()
            
        elif isfile(cg):
       
            fs=fileSoup(cg)
            fa=fs.find_all('a')
            rf=(e.get('href') for e in fa)    
            jo=(urljoin(url,e) for e in rf if e!=None)
            rs=singli(jo)
            s.figs2frame(rs)
            s.figs=rs            
            
        else:
            showerror('Erro','"%s"\nNão é um diretório válido'%cg)

    def scanDir(s,dr):

        s.path='%s:/'%dr
        if s.path in s.datas:
            s.data=s.datas[s.path]
            s.makeTree()
        else:
            s.scanPath()

    def scanPath(s):        

        if isdir(s.path):
            s.tit.config(text=unquote(s.path))           

            s.processData()            
            s.data=tuple(s.dti.values())
            s.datas[s.path]=s.data

            s.salvarConfigs()

    def selecionarTudo(s):

        if s.lift=='tree':
        
            chs=s.tree.get_children()
            s.tree.selection_set(chs)

        elif s.lift=='tred':
        
            chs=s.tred.get_children()
            s.tred.selection_set(chs)

        elif s.lift=='text':

            s.tex.focus()
            s.tex.tag_add('sel','1.0',END)

    def selecionarNada(s):

        if s.lift=='tree':
            
            tis=s.tree.selection()
            s.tree.selection_remove(tis)

        elif s.lift=='tred':
            
            tis=s.tred.selection()
            s.tred.selection_remove(tis)

        elif s.lift=='text':
            
            s.tex.tag_remove(SEL, "1.0", END)
            
        
    def salvarConfigs(s):

        try:
            f=open('favoritHist.db','wb')
            dump((s.favoritHist,),f)
            f.close()
            
        except Exception as ex:
            printex('favoritHist.db',ex)
            
        try:
            f=open('dirsHist.db','wb')
            dump((s.dirsHist,),f)
            f.close()
            
        except Exception as ex:
            printex('dirsHist.db',ex)
           
        try:
            f=open('tfsConfigs.db','wb')
            dump((s.path,s.data,s.datas),f)
            f.close()
            
        except Exception as ex:
            printex('tfsConfigs.db',ex)           
            
        try:
            di=list(s.filesHist.items())
            s.filesHist=dict(di[-50:])
            f=open('filesHist.db','wb')
            dump((s.filesHist,),f)
            f.close()
            
        except Exception as ex:
            printex('filesHist.db',ex)
            
        try:
            di=list(s.urlsHist.items())
            s.urlsHist=dict(di[-50:])
            f=open('urlsHist.db','wb')
            dump((s.urlsHist,),f)
            f.close()
            
        except Exception as ex:
            printex('urlsHist.db',ex)          
            
        try:
            f=open('expandHist.db','wb')
            dump((s.expandHist,),f)
            f.close()
            
        except Exception as ex:
            printex('expandHist',ex)
           

        try:
            f=open('TFS/tfsConfigs.db','wb')
            dump((s.path,s.data,s.datas),f)
            f.close()
            
        except Exception as ex:
            print('TFS/tfsConfigs.db')
            print(ex)
            print()

        try:
            f=open('fgPt.db','wb')
            dump((s.fgPt,),f)
            f.close()

        except Exception as ex:
            printex('fgPt.db',ex)           

        try:
            f=open('fgTmIm.db','wb')
            dump((s.fgTmIm,),f)
            f.close()

        except Exception as ex:
            printex('fgTmIm.db',ex)           

    def lembrarConfigs(s):

        try:
            f=open('favoritHist.db','rb')
            ld=load(f)
            f.close()
            s.favoritHist=ld[0]         
            
        except Exception as ex:            
            
            s.favoritHist=dict()

        try:
            f=open('dirsHist.db','rb')
            ld=load(f)
            f.close()
            s.dirsHist=ld[0]         
            
        except Exception as ex:
            s.dirsHist=dict()

        try:
            f=open('tfsConfigs.db','rb')
            ld=load(f)
            f.close()
            
            s.path=ld[0]
            s.data=tuple([e for e in  ld[1] if exists(e[0])])          
            s.datas=ld[2]            
            
        except Exception as ex:            
            s.path=''
            s.data=tuple([])
            s.datas=dict()

        try:
            f=open('filesHist.db','rb')
            ld=load(f)
            f.close()
            s.filesHist=ld[0]
            di=list(s.filesHist.items())[-50:]
            s.filesHist=dict(di)
              
            
        except:
            s.filesHist=dict()

        try:
            
            f=open('urlsHist.db','rb')
            ld=load(f)
            f.close()
            s.urlsHist=ld[0]
            di=list(s.urlsHist.items())[-50:]
            s.urlsHist=dict(di)

        except:
            s.urlsHist=dict()
            

        try:
            f=open('expandHist.db','rb')
            ld=load(f)
            f.close()
            s.expandHist=ld[0]        
            
        except:
            s.expandHist=dict()

        try:
            f=open('fgPt.db','rb')
            ld=load(f)
            f.close()
            s.fgPt=ld[0]
        except:
            s.fgPt=dict()

        try:
            f=open('fgTmIm.db','rb')
            ld=load(f)
            f.close()
            s.fgTmIm=ld[0]
        except:
            s.fgTmIm=dict()
            
    def telaInicial(s):        

        canvas=Canvas(s.ms)
        grid1(canvas)

        dr='D:/Wallpers'
        
        if isdir(dr):
            fs=files(dr)
            fe=filtrAny(fs)
            fe=[fg for fg in fe if isImage(fg)]
            fl=choice(fe)
            im=Image.open(fl)
            cw,ch=wigSize(s.canvas)
            iw,ih=im.size
            
            if cw>ch:
                sz=fitWidth(cw,ch,iw,ih)
            else:
                sz=fitHeight(cw,ch,iw,ih)
                
            im=im.resize(sz)
            iw,ih=im.size
            
            x,y=wigCenter(s.canvas)
            s.ph=ImageTk.PhotoImage(im)            
            canvas.create_image(x,y,image=s.ph)
            canvas['scrollregion']=(0,0,0,0)
            canvas.bind('<Control-space>',Fc(s.liftAnterior))
            canvas.focus_force()

            s.fig=fl
            s.img=Image.open(s.fig)

    def refreshFolder(s):               
            

        if isdir(s.path):
            s.processData()
            s.rev=1
            s.sortColumn(u'Data_modificação')
            

    def addDir(s,dr):
        
        try:
                
            bn=basename(dr)
            dn=dirname(dr)
            ls=folderLen(dr)        
            sz=folderSize(dr)
            mt=getMtime(dr)
            bs=''
            
            tg='dr'

            ff=folderFiles(dr)
            df=dict()
            
            for e in ff:
                
                ex=splitext(e)[-1].lower()
                if len(ex)<=8:
                    
                    if ex in df:
                        df[ex]+=1
                    else:
                        df[ex]=1

                    if ex in s.dfe:
                        s.dfe[ex]+=1
                    else:
                        s.dfe[ex]=1
            
            di=df.items()        
            di=[(b,a) for (a,b) in di]
            
            di.sort()        
            di=di[::-1]

            ts=[]
            for (v,e) in di:
                if e in s.dbs:
                    s.dbs[e]+=v
                else:
                    s.dbs[e]=v
                tx='  %d%s'%(v,e)
                ts.append(tx)
            bs='  '.join(ts)

            item=(bn,'',ls,sz,mt,bs,dn,tg)

            s.aqs+=ls
            s.ttt+=sz
            ftt=formatedSize(s.ttt)

            fz=formatedSize(sz)
            ft=amd2dma(mt)

            vals=(bn,'',ls,fz,ft,bs)
            
            res=[dr,item,vals]
            if not res in s.drVals:
                s.drVals.append(res)

        except Exception as exc:
            printex('addDir',exc)
            

    def addFile(s,fl):

        try:
           
            bn=basename(fl)
            dn=dirname(fl)
            ex=splitext(bn)[1]           
            
            sz=getsize(fl)
            mt=getMtime(fl)
            tg='fl'
            df=None
            s.aqs+=1

            fs=1
            bs=''

            if ex=='.zip':
               
                try:                    
                    zf=ZipFile(fl)
                    nl=zf.namelist()
                    fs=len(nl)
                    zf.close()
                except Exception as exc:
            
                    print('em: addFile')
                    print(fl)
                    print(exc)


            elif ex in dkvExt:

                try:
                
                    kv=openDkv(fl)
                    fs=len(kv)
                    fs=len(list(kv.keys()))

                except Exception as exc:
                    print('em addfile - ex in dkv')
                    print(fl)
                    print(exc)
                    print()

            elif ex in dataExt:
                try:
                    df=s.dataRead(fl)
                    fs=len(df)
                except:                        
                    pass
          
            elif ex in ['.txt','.js','.css','.html','.xml']:
                try:
                    tl=textLines(fl)
                    fs=len(tl)
                except:
                    fs=1


            elif ex in ['.uvs']:
                
                
                kv=uvs2dkv(fl)
                fs=len(kv)
                    
            item=[bn,ex,fs,sz,mt,bs,dn,tg]

            fz=formatedSize(sz)
            ft=amd2dma(mt)
            if bs==0:
                bs=''
            vals=[bn,ex,fs,fz,ft,bs]

            s.flVals.append([fl,item,vals])

        except Exception as exc:
            print(exc)
              
    def parar(s):

        s.stop=1        
        s.stb.config(text='Processo abortado')

    def scanAllDrivers(s):

        for dv in list('defghijklmnopqrstuvwxyz'.upper()):            
            s.path='%s:/'%dv
            if isdir(s.path):
                s.tit.config(text=s.path)
                s.processData()

        for dv in list(s.favoritHist.keys()):
            s.path=dv
            if isdir(s.path):
                s.tit.config(text=s.path)
                s.processData()
        
    def processData(s):
        
        s.delTree()        
        s.tree.update()
        s.liftWidget('tree') 

        s.tit.config(text=unquote(s.path))
        
        s.aqs=0
        s.ttt=0

        s.dtx=dict()
        s.dti=dict()
        s.dxt=dict()

        #addDirs

        no=[
            'Config.Msi',
            'Recovery',
            'Arquivos de Programas',
            'Documents and Settings',            
            'System Volume Information',
            'C:/Windows',
            '$',
            ]
        
        ds=list(listds(s.path))
        ds=[dr for dr in ds if not containAny(dr,no)]            
        
        lds=len(ds)
        
        s.dfe=dict()

        s.stop=0
        ld=0
        s.dbs=dict()
        for ic,dr in enumerate(ds):            
            if s.stop==1:
                break
            s.drVals=[]
            try:
                s.addDir(dr)
            except:
                pass

            for dr,item,vals in s.drVals:
                
                ti=s.tree.insert('','end',values=vals,tags='dr')
                
                s.dtx[ti]=dr
                s.dti[ti]=item
                s.dxt[dr]=ti
                ld+=1

            tx='%d Pastas de %d'%(ld,lds)
            
            s.stb.config(text=tx)
            s.stc.config(text='')
            s.tree.yview('moveto',1)                

            s.tree.tag_configure('dr',foreground='black',)
            s.tree.tag_configure('fl',foreground='black')

            s.ms.update()
        
        #addFiles

        s.aqs=0
            
        fs=list(listfs(s.path))
        np=npartit(fs[:10])
        
        ps=npartit(fs[10:],500)
        np.extend(ps)
        lfs=len(fs)
        
        s.figs=[]
        s.orig=s.figs[:]

        s.tree.tag_configure('fl',foreground='black')

        s.stop=0
        lf=0
        for ic,pt in enumerate(np):           
            
            if s.stop==1:
                break
            s.flVals=[]            
            concurrent(s.addFile,pt)

            for fl,item,vals in s.flVals:
                
                if containAny(fl,imagExt):                    
                    s.figs.append(fl)

                elif endsAny(fl.lower(),dataExt):
                    try:
                        df=s.dataRead(fl)
                    except:                        
                        continue
                     
                    ls=len(df)
                    item[2]=ls
                    vals[2]=ls

                    bs='%d.cols'%len(df[0])
                    vals[5]=bs
                    item[5]=bs
                    
                ti=s.tree.insert('','end',values=vals,tags='fl')
            
                s.dtx[ti]=fl
                s.dti[ti]=item
                s.dxt[fl]=ti
                lf+=1
                
            s.tree.yview('moveto',1)
            tx='%d de %d'%(lf,lfs)
            s.stb.config(text=tx)
            s.ms.update()        
            
        s.data=tuple(s.dti.values())         
        s.datas[s.path]=s.data
        
        try:
            s.salvarConfigs()
        except Exception as ex:
            print('addFiles')
            print(ex)
            print()

        vs=tuple(s.dti.values())
        tt=sum([e[3] for e in vs])
        tz=formatedSize(tt)
        
        lv=tuple(s.dti.values())
        ss=[et[3] for et in lv]
        su=sum(ss)
        fz=formatedSize(su)
        
        tx='%d Pastas de %d e %d Arquivos de %d - Em %s'%(ld,lds,lf,lfs,tz)
        
        s.stb.config(text=tx)
        
        s.rev=0
       
        s.tree.tag_configure('dr',foreground='blue',)
        s.tree.tag_configure('fl',foreground='purple')
        s.tree.yview('moveto',0)
        s.salvarConfigs()
        
        lx=list(s.dxt.keys())
        s.files=[e for e in lx if isfile(e)]
        s.figs=filtrAny(s.files)
        s.orig=s.figs[:]

    def copiarTexto(s):

        if s.lift=='tree':        

            tis=s.tree.selection()
            ens=[s.dtx[ti] for ti in tis]
            tx='\n'.join(ens)

        elif s.lift=='tred':
            
            tis=s.tred.selection()
            try:
                t1=s.tred.item(tis[1],'values')            
                ic=listind(t1,'http')
            except:
                to=s.tred.item(tis[0],'values')
                ic=listind(to,'http')
            ens=[s.tred.item(ti,'values')[ic] for ti in tis]
            tx='\n'.join(ens)
            
        elif s.lift=='text':            

            try:
                tx=s.tex.selection_get()
            except:
                tx=s.tex.get('0.0',END)

        elif s.lift=='frame':

            tx=s.tit['text']

        elif s.lift=='canvas':

            tx=s.fig

        elif s.lift=='box':

            sel=s.box.curselection()
            li=[s.box.get(e) for e in sel]
            tx='\n'.join(li)

        else:
            tx=s.path

        s.ms.clipboard_clear()
        s.ms.clipboard_append(tx)

        sp=tx.split('\n')
        if len(sp)>=10:
            tx='\n'.join(sp[:5]+['---------']+sp[-5:])

        tp=Toplevel()
        lb=Label(tp,
                 text='Copiado:\n%s\n'%tx,
                 fg='red',
                 font=('Times',12),
                 )
        lb.grid(padx=20, pady=20)
        tp.update()
        centralizar(tp)
        tp.after(3000,Fc(tp.destroy))

        return tx

    def verTreeFocus(s):

        ti=s.tree.focus()
        s.tree.see(ti)            

    def lancarArquivo(s):        

        if s.lift=='text':
            try:
                le=s.getTextLine()
                os.startfile(le)
            except:
                os.startfile(s.path)

        elif s.lift=='tree':

            ti=s.tree.focus()
            bn=s.tree.item(ti)['values'][0]
            bn=quote(bn)
            fl=pjoin(s.path,bn)
                
            if isfile(fl):               
                
                if fl.endswith(('.dkv',)):
                    dc=openDkv(fl)
                    di=list(dc.items())
                    li=[]
                    for ky,vs in di:
                        li.append(ky)
                        li.extend(vs)
                        li.append('\n')
                    tx='\n'.join(li)
                    s.wTx.settext(tx)
                    s.liftWidget('text')

                else:                    
                    os.startfile(fl)
            else:                    
                ur=nm2ur(bn)
                for et in['.txt','.dkv','.zip']:
                    ur=ur.replace(et,'')
                os.startfile(ur)

        elif all([s.lift=='tred',len(s.rzn)>0]):

            ti=s.tred.focus()
            fn=s.tred.item(ti)['values'][0]

            s.rzf.extract(fn,'temp')
            cd=os.getcwd()
            fl=rpath('%s/temp/%s'%(cd,fn))
            os.startfile(fl)            

            if all([fn in s.rzn,containAny(fn,imagExt)]):
                rd=s.rzf.read(fl)
                ib=io.BytesIO(rd)        
                im=Image.open(ib)
                s.toCanvas(rd,im)               
                
            else:
                s.rzf.extract(fn,'temp')
                cd=os.getcwd()
                fl=rpath('%s/temp/%s'%(cd,fn))
                os.startfile(fl)

        elif s.lift=='frame':
            
            fl=s.path
            if fl.endswith(('.dkv',)):
                dc=openDkv(fl)
                di=list(dc.items())
                li=[]
                for ky,vs in di:
                    li.append(ky)
                    li.extend(vs)
                    li.append('\n')
                tx='\n'.join(li)
                s.wTx.settext(tx)
                s.liftWidget('text')

            else:                    
                os.startfile(fl)           


        elif s.lift=='canvas':
            fl=s.path
            if not fileExt(fl) in zipExt:
                os.startfile(fl)
           
    def copiarPara(s):
        
        if s.lift=='tree':

            tis=s.tree.selection()
            dst=askdirectory(initialdir=s.path)            

            if dst:                
                    
                for ti in tis:                   
                    
                    fx=s.dtx[ti]
                    os.chmod(fx,stat.S_IREAD)
                                     
                    if isfile(fx):
                        shutil.copy2(fx,dst)
                        
                    elif isdir(fx):                       
                            
                        ff=folderFiles(fx)
                        
                        for et in ff:
                            
                            et=rpath(et)
                            md='/'.join(et.split('/')[1:-1])
                            
                            pj=rpath(pjoin(dst,md))                            
                            if not isdir(pj):
                                os.makedirs(pj)
                                
                            os.chmod(et,stat.S_IREAD)
                            os.chmod(pj,stat.S_IWRITE)
                            
                            shutil.copy(et,pj)

                scs=[s.dtx[ti] for ti in tis]
                
                if len(scs)>10:
                    scs=scs[:10]+['--------']+scs[-10:]
                    
                tx='\n'.join(scs)
                showinfo('Copiado',
                         'Copiado de:\n%s\n\npara:\n%s'%(tx,dst))
                
        if s.lift=='frame':
                        
            dst=askdirectory(initialdir=dirname(s.path))
            
            if dst:

                if not isdir(dst):
                    os.makedirs(dst)
                    
                if isfile(s.path):
                    shutil.copy2(s.path,dst)
                    
                elif all([isdir(s.path),len(s.figs)>0]):
                    shutil.copytree(gt,dst)
                

        if s.lift=='canvas':

            bn=basename(s.fig)
            dst=asksaveasfilename(initialdir=s.path,
                                initialfile=bn)
            if dst:                
                
                if isImage(s.fig):
                    s.img.save(dst)

                elif isUi(s.fig):

                    rc=reqContent(s.fig)
                    ib=io.BytesIO(rc)
                    im=Image.open(ib)
        
                    gv=ib.getvalue()            
            
                    op=open(dst,'wb')
                    op.write(gv)
                    op.close()
                    
                elif s.fig in s.rzf.namelist():                    
                    rd=s.rzf.read(bn)
                    with open(dst, 'wb') as f:
                        f.write(rd)

                elif all([isUi(s.fig),splitext(s.path)[1]=='.txt']):
                    urlRetrieve(s.fig,'temp')
                    shutil.copy(pjoin(cd,'temp'),dst)
                    
        if s.lift=='text':

            sv=asksaveasfilename()
            if sv:

                with open(sv,'w') as dst:
                    sel=s.tex.get(SEL_FIRST, SEL_LAST)
                    dst.write(sel)

    def browseMaxDims(s):

        cd=os.getcwd()
        bn=basename(s.fig)
        dst=rpath(pjoin(cd,'temp/%s'%bn))
        s.img.save(dst)            
        s.toHtml(dst)        

    def moverPara(s):

        if s.lift=='tree':        
            tis=s.tree.selection()
            sv=askdirectory()
            
            if sv:

                if not isdir(sv):
                    os.makedirs(sv)
                
                for ti in tis:
                    
                    fl=s.dtx[ti]
                    bn=basename(fl)
                    dn=dirname(fl)
                    
                    if isfile(fl):                        
                       
                        it=s.dti[ti]                        
                        ds=pjoin(sv,bn)                        
                        shutil.move(fl,ds)                                       
    
                    elif isdir(fl):
                        
                        ds=pjoin(sv,bn)
                        shutil.copytree(fl,ds)
                        s.toTrash(fl)                

                    s.tree.delete(ti)
                    
                s.datas[dn]=tuple(s.dti.values())
                    

    def removerDoDisco(s):

        if all([s.lift=='text',splitext(s.path)[1]=='.txt']):
            gts=s.getSel()            
            
            if len(gts)<10:
                tx='\n'.join(gts)                
            else:
                tx='\n'.join(gts[:5]+['----------']+gts[-5:])
            
            ak=askokcancel('Remover',
                           'Tem certeza que quer remover\n'
                           'permanentemente do disco\n'
                           '"%s?"'%tx)

            if ak:
                tl=textLines(s.path)                
                nt='\n'.join(list(set(tl)-set(gts)))
                with open(s.path,'w') as ds:
                    ds.write(nt)
                s.mostrarTexto(s.path)
            

        elif all([s.lift=='tree',splitext(s.path)[1] in ['.zip','.rar']]):
        
            ti=s.tree.selection()[0]
            df=s.dtx[ti]
            nex=near(s.tree.get_children(),ti)           

            tis=s.tree.get_children()

            ak=askokcancel('Remover',
                           'Tem certeza que quer remover\n'
                           'permanentemente do disco\n'
                           '"%s?"'%df)

            if ak:

                try:
                    s.rzf.close()
                except:
                    pass
                
                if isfile(df):
                    
                    os.chmod(df,stat.S_IWRITE)
                    s.toTrash(df)
                    it=s.dti[ti]
                    
                    if df in s.dfs:
                        s.dfs.remove(df)
                    if df in s.figs:
                        s.figs.remove(df)
                    
                elif isdir(df):
                    os.chmod(df,stat.S_IWRITE)
                    s.toTrash(df)
                    it=s.dti[ti]            
                    
                s.tree.delete(ti)
                dn=dirname(df)
                dt=s.datas[dn]
                
                
                dat=[et for et in list(s.dti.values()) if et!=ti]
                s.datas[s.path]=dat

                s.canvas.delete(ALL)
                s.clearFrame()
                
                s.tree.selection_remove(ti)
                
                s.tree.selection_set(nex)
                s.tree.focus(nex)
                s.tree.focus_set()
                s.tree.see(nex)          

        elif all([s.lift=='tree', fileExt(s.path) not in zipExt]):
        
            tis=s.tree.selection()
            dfs=[s.dtx[ti] for ti in tis]            

            exc=dfs[:5]+['-------------']+dfs[-5:]
            
            ak=askokcancel('Remover',
                           'Tem certeza que quer remover\n'
                           'permanentemente do disco\n'
                           '"%s?"'%'\n'.join(exc))

            zns=zip(tis,dfs)
            
            if ak:

                for ti,df in zns:
                    
                    if isfile(df):
                        try:
                            s.rzf.close()
                        except:
                            pass

                        os.chmod(df,stat.S_IWRITE)
                        os.remove(df)
                        it=s.dti[ti]
                        if df in s.figs:
                            s.figs.remove(df)
                        
                    elif isdir(df):
                        os.chmod(df,stat.S_IWRITE)
                        shutil.rmtree(df)
                        it=s.dti[ti]

                    tis=s.tree.get_children()
                        
                    s.tree.delete(ti)

                    dn=dirname(df)
                    dt=s.datas[dn]
                    dt=tuple([e for e in dt if e!=s.dti[ti]])

                dat=[et for et in list(s.dti.values()) if not et in exc]
                s.datas[s.path]=dat

                s.canvas.delete(ALL)
                s.clearFrame()
                s.tree.selection_clear()
                s.tree.focus_set()

        elif s.lift=='frame' and s.path.endswith(('.txt','.dkv','.udc','.uvs')):

            tis=s.tred.get_children()
            ti=s.tred.focus()            
           
            ky=s.dtk[ti]
            ak=askokcancel('Remover',
                           'Tem certeza que quer remover\n'
                           'permanentemente do disco\n'
                           '"%s?"'%ky)
            if ak:

                ic=tis.index(ti)
                ic+=1
                nx=tis[ic]
                
                s.tred.selection_clear()
                s.tred.selection_set(nx)
                s.tred.focus(nx)
                s.fromTred()
                
                s.tred.delete(ti)
                del s.dkv[ky]
                saveDkv(s.dkv,s.path)
                   

        elif s.lift=='tred' and s.path.endswith(('.txt','.dkv','.udc','.uvs')):

            tis=s.tred.selection()
            fns=[s.tred.item(ti,'values')[0] for ti in tis]
            exc=fns[:5]+['-------------']+fns[-5:]
            
            ak=askokcancel('Remover',
                           'Tem certeza que quer remover\n'
                           'permanentemente do disco\n'
                           '"%s?"'%'\n'.join(exc))
            '''
            s.dtd[ti]=item 
            s.dtf[ti]=fn
            s.ddt[fn]=ti
            '''

            if ak:

                if s.path.endswith(('.dkv','.udc')):
                    for ti in tis:
                        s.tred.delete(ti)                    

                    for fn in fns:
                        del s.dkv[fn]
                    bn,ex=splitext(s.path)
                    
                    dst=bn+'.dkv'
                    saveDkv(s.dkv,dst)

                    s.data=tuple(s.dtd.values())
                    s.datas[s.path]=s.data
                    

                elif s.path.endswith(('.txt',)):

                    rms=[]
                    for fn in fns:
                        rms.extend(list(s.dkv[fn]))
                        del s.dkv[fn]
                        s.tred.delete(tis)
                    
                    tls=list(set(s.textlines)-set(rms))
                    tls=[et for et in tls if et!='']
                    
                    op=open(s.path,'w')
                    tx='\n'.join(tls)
                    op.write(tx)
                    op.close()                
                    s.stb.config(text=len(tls))

                elif s.path.endswith(('.uvs',)):

                    rms=[]
                    for fn in fns:
                        
                        rms.extend(list(s.dkv[fn]))
                        del s.dkv[fn]
                        s.tred.delete(tis)
                    
                    di=s.dkv.items()
                    tls=[]
                    kys=0
                    vls=0
                    for ky,vs in di:
                        tls.append(ky)
                        tls.append('\n')
                        tls.extend(vs)
                        kys+=1
                        vls+=len(vs)                            
                    op=open(s.path,'w')
                    tx='\n'.join(tls)
                    op.write(tx)
                    op.close()                
                    s.stb.config(text='Keys: %d - vls: %d'%(kys,vls))

        elif all([s.lift=='tred',fileExt(s.path) in dataExt]):
            
            tis=s.tred.selection()
            
            for ti in tis:
                s.tred.delete(ti)

            s.clearEntry()            
            s.ms.update()            
            s.saveFromTred()

        elif all([s.lift=='tred','Histórico de urls' in s.sta['text']]):

            ti=s.tred.focus()            
            vs=s.tred.item(ti,'values')
            ic=listind(vs,'http')
            ur=vs[ic]
            del s.urlsHist[ur]
            s.tred.delete(ti)
            s.verUrlsHist()

        elif all([s.lift=='tred','Histórico de favoritos' in s.sta['text']]):

            ti=s.tred.focus()            
            vs=s.tred.item(ti,'values')
##            ic=listind(vs,'http')
            dr=vs[1]
            del s.favoritHist[dr]
            s.tred.delete(ti)
            
            

        elif all([s.lift=='tred',fileExt(s.path) in zipExt]):

            ti=s.tred.focus()            
            s.fig=s.dtd[ti]
            tis=s.tred.get_children()

            ak=askokcancel('Remover',
                       'Tem certeza que quer remover\n'
                       'permanentemente do disco:\n'
                       '"%s?"'%s.fig)

            if ak:
                
                nex=near(tis,ti)
                s.tred.delete(ti)
                
                nm,ex=splitext(s.zip)
                sn=nm+'.bak'+ex                
                s.rzf.close()
                
                os.rename(s.zip,sn) 
                sz=ZipFile(sn,'r')
                rl=[fg for fg in sz.namelist() if fg!=s.fig]
                
                dz=ZipFile(s.zip,'w')               
            
                for fn in rl:        
                    try:            
                        rf=sz.read(fn)
                        ib=io.BytesIO(rf)
                        gv=ib.getvalue()
                        dz.writestr(fn,gv)
                    except Exception as ex:
                        print('removerDoDisco')
                        print(fn)
                        print(ex)
                        print()
                        
                sz.close()
                dz.close()

                os.remove(sn)

                s.tred.selection_clear()
                s.tred.selection_set(nex)
                s.tred.see(nex)
                s.tred.focus(nex)
                

                dnp=dirname(s.path)                
                rows=list(s.datas[dnp])
                
                ti=s.tree.focus()
                item=s.dti[ti]                
                rows.remove(item)
                
                s.salvarConfigs()
                s.datas[dnp]=tuple(rows)
                s.tree.event_generate('<F5>')           
            

        elif all([s.lift=='canvas',isfile(s.fig)]):
            
            ak=askokcancel('Remover',
                       'Tem certeza que quer remover\n'
                       'permanentemente do disco:\n'
                       '"%s?"'%s.fig)

            if ak:

                df=s.fig
                s.canvas.delete('img')
                s.img.close()
                
                s.toTrash(s.fig)
                s.figs.remove(s.fig)
                
                ti=s.tree.focus()                
                ic=tis.index(ti)
                s.tree.delete(ti)

                dn=dirname(df)
                dt=s.datas[dn]                
                
                wc=wigCenter(s.canvas)
                s.canvas.create_text(wc,
                                     text='REMOVIDO',
                                     fill='red',
                                     font=('Arial',30,'bold'))

                dnp=dirname(s.path)                
                rows=list(s.datas[dnp])                
                item=s.dti[ti]                
                rows.remove(item)
                s.salvarConfigs()
                s.datas[dnp]=tuple(rows)
                s.tree.event_generate('<F5>')

                t2=near(tis,ti)
                
                s.tree.selection_clear()
                s.tree.selection_set(t2)
                s.tree.focus(t2)
                s.tree.see(t2)
               
                tf=s.tree.focus()              
                s.tree.event_generate('<Return>')
                
        
        elif all([s.lift=='canvas',s.fig in s.rzn]):           
            
            ak=askokcancel('Remover',
                       'Tem certeza que quer remover\n'
                       'permanentemente do disco\n'
                       '"%s?"'%s.fig)

            if ak:

                nm,ex=splitext(s.zip)
                sn=nm+'.bak'+ex                
                s.rzf.close()
                
                os.rename(s.zip,sn) 
                sz=ZipFile(sn,'r')
                rl=[fg for fg in sz.namelist() if fg!=s.fig]
                
                dz=ZipFile(s.zip,'w')               
            
                for fn in rl:        
                    try:            
                        rf=sz.read(fn)
                        ib=io.BytesIO(rf)
                        gv=ib.getvalue()
                        dz.writestr(fn,gv)
                    except Exception as ex:
                        print(fn)
                        print(ex)
                        print()
                        
                sz.close()
                dz.close()

                os.remove(sn)
                s.mostrarZip(s.zip)               
        
        elif all([s.lift=='frame',fileExt(s.path) in zipExt]):

            tis=s.tree.get_children()
            ak=askokcancel('Remover',
                       'Tem certeza que quer remover\n'
                       'permanentemente do disco\n'
                       '"%s?"'%s.path)

            if ak:
            
                ti=s.tree.selection()[0]
                df=s.dtx[ti]
                nex=near(tis,ti)   

                try:
                    s.rzf.close()
                except:
                    pass
                
                if isfile(df):
                    os.chmod(df,stat.S_IWRITE)
                    s.toTrash(df)
                    it=s.dti[ti]
                    if df in s.dfs:
                        s.dfs.remove(df)
                    if df in s.figs:
                        s.figs.remove(df)
                    
                elif isdir(df):
                    os.chmod(df,stat.S_IWRITE)
                    s.toTrash(df)
                    it=s.dti[ti]            
                    
                s.tree.delete(ti)
                dn=dirname(df)
                dt=s.datas[dn]

                s.canvas.delete(ALL)
                s.clearFrame()
                
                s.tree.selection_clear()
                
                s.tree.selection_set(nex)
                s.tree.focus(nex)
                s.tree.focus_set()
                s.tree.see(nex)          
                rz=s.dtx[nex]
                s.path=rz
                s.mostrarZip(rz)

        elif all([s.lift=='tred','expandidas' in s.sta.cget('text')]):

            ti=s.tred.focus()            
            vs=s.tred.item(ti,'values')
            try:
                ic=listind(vs,'http')
            except:
                ic=1
            et=vs[ic]
            del s.expandHist[et]
            s.salvarConfigs()
                
            s.verExpandHist()

        else:            

            showinfo('Info','ant: %s\npath: %s\nlift: %s\n'%(s.ant,s.path,s.lift))            
       

    def fromEntry(s,ev):
        print(ev)
        print()
        print(dir(ev))
        print(help(ev))
        print()
        li=['char', 'delta', 'height', 'keycode',
            'keysym', 'keysym_num', 'num', 'send_event',
            'serial', 'state', 'time', 'type', 'widget',
            'width', 'x', 'x_root', 'y', 'y_root']
        for et in li:
            print('%s.%s'%(ev,et))
            print()
        s.ev=ev


        sv=s.sve.get()
        
        if urOk(sv):
            url=sv
            if isUi(url):
                s.fig=url
                s.verNoBrowser()
            else:
                s.expandirUrl(url)
        else:
            s.filtrar()

    def filtrar(s):       
        
        ge=s.sve.get()       
         
        if s.lift=='tree':

            tis=s.dti.keys()
            if ge=='':                
                for ti in tis:                    
                    s.tree.reattach(ti,'','end')
            
            gs=ge.split()
            gs=[e.lower() for e in gs]                   
                
            bns=list(s.dtx.values())            
            
            if s.filt=='All':
                fa=filtrAll(bns,gs)
                
            elif s.filt=='Any':
                fa=filtrAny(bns,gs)                
                
            elif s.filt=='End':
                fa=filtrEnd(bns,gs)
                
            elif s.filt=='Start':
                fa=filtrStart(bns,gs)
                
            zp=zip(tis,bns)

            for ti in tis:
                s.tree.reattach(ti,'','end')
            
            for ti,vl in zp:
                if not vl in fa:
                    s.tree.detach(ti)
                    
            tis=s.tree.get_children()
            s.files=[s.dtx[ti] for ti in tis]
            s.files=[e for e in s.files if isfile(e)]
            s.figs=filtrAny(s.files)
            s.orig=s.figs[:]

            lb=len(bns)
            lf=len(tis)
            s.stb.config(text='%d de %d'%(lf,lb))

            
        elif s.lift=='tred':

            tis=list(s.dtd.keys())
            bns=s.dtd.values()
            
            if ge=='':
                for ti in tis:
                    try:
                        s.tred.reattach(ti,'','end')
                    except:
                        pass
                  
            gs=ge.split()
            
            if s.filt=='All':
                fa=filtrAll(bns,gs)
                
            elif s.filt=='Any':                
                fa=filtrAny(bns,gs)
                
            elif s.filt=='End':
                fa=filtrAny(bns,gs)
                
            elif s.filt=='Start':
                fa=filtrStart(bns,gs)

            zp=zip(tis,bns)
            
            for ti,vl in zp:
                if not vl in fa:
                    try:
                        s.tred.detach(ti)
                    except:
                        pass

            try:
                tis=s.tred.get_children()
                t1=tis[1]
                vls=s.tred.item(t1,'values')
                ic=listind(vls,'http')
                s.figs=[s.tred.item(ti,'values')[ic] for ti in tis]
            except:
                pass
      

            lb=len(bns)
            lf=len(tis)
            s.stb.config(text='%d de %d'%(lf,lb))

        elif s.lift=='text':

            if ge=='':
                s.mostrarTexto(s.path)
                
            gs=ge.split()

            if s.filt=='All':
                fa=filtrAll(s.textlines,gs)
                
            elif s.filt=='Any':
                fa=filtrAny(s.textlines,gs)
                
            elif s.filt=='End':
                fa=filtrAny(s.textlines,gs)
                
            elif s.filt=='Start':
                fa=filtrStart(s.textlines,gs)
            
            tx='\n'.join(fa)
            s.wTx.settext(tx)                
           
            s.figs=filtrAny(fa)
            s.orig=s.figs[:]
            ll=len(s.textlines)
            lf=len(s.figs)
            tx='Lines: %d   Figs: %d'%(ll,lf)
            s.stb.config(text=tx)

        elif s.lift=='box':                

            gs=ge.split()
            ga=s.box.get(0,END)

            if s.filt=='All':
                fa=filtrAll(ga,gs)
                
            elif s.filt=='Any':
                fa=filtrAny(ga,gs)
                
            elif s.filt=='End':
                fa=filtrAny(ga,gs)
                
            elif s.filt=='Start':
                fa=filtrStart(ga,gs)
            
            s.wBx.setlist(fa)
            s.stb.config(text=s.box.size())

    def clearEntry(s):

        s.sve.set('')
        s.filtrar()

    def clearFrame(s):

        chs=s.frame.winfo_children()
        for ch in chs:
            ch.destroy()
        s.frame.focus_force()

    def clearTree(s):
 
        chs=s.tree.get_children()
        for ch in chs:
            s.tree.detach(ch)
        s.ms.update()

    def delTree(s):

        chs=s.tree.get_children()
        for ch in chs:
            s.tree.delete(ch)
        s.ms.update()        

    def makeTree(s):

        s.liftWidget('tree')

        s.dti=dict()
        s.dtx=dict()
        s.dxt=dict()

        s.delTree()        
        s.tit.config(text=unquote(s.path))

        dc={}
        for et in s.data:
            x=pjoin(et[-2],et[0])
            if any([isdir(x),isfile(x)]):
                dc[et[0]]=et
        s.data=tuple(dc.values())
        
        for item in s.data:         

            bn,ex,fs,sz,mt,bs,dn,tg=item
            fl=pjoin(dn,bn)
            
            mt=invDat(mt)
            fz=fmSize(sz)

            if ex==0:
                ex=''
            
            vals=[bn,ex,fs,fz,mt,bs]            
            ti=s.tree.insert('','end',values=vals,tag=tg)
            
            s.dti[ti]=item
            s.dtx[ti]=fl
            s.dxt[fl]=ti            

        s.rev==0

        lx=list(s.dxt.keys())
        
        s.dirs=[e for e in lx if isdir(e)]
        s.files=[e for e in lx if isfile(e)]        
        
        s.zips=filtrAny(s.files,zipExt)
        s.figs=filtrAny(s.files,imagExt)
        
        s.orig=s.figs[:]
        s.lift='tree'

        ld=len(s.dirs)
        lf=len(s.files)        
        tz=sum([e[3] for e in s.data])
        fz=formatedSize(tz)

        try:
            tx='%d Pastas e %d Arquivos - Em %s'%(ld,lf,fz)
        except:
            tx='%d Pastas e %d Arquivos'%(ld,lf)            
        
        s.stb.config(text=tx)

        s.tree.tag_configure('dr',foreground='blue',)
        s.tree.tag_configure('fl',foreground='purple')

        try:
            s.sortColumn(u'Data_modificação')
        except:
            pass

        
     


    def misturarColunas(s):

        if s.lift=='tree':

            tis=list(s.tree.get_children())
            shuffle(tis)

            ax=[]
            for ti in tis:
                s.tree.reattach(ti,'',END)
                

        elif s.lift=='tred':

            tis=list(s.tred.get_children())
            shuffle(tis)

            ax=[]
            for ti in tis:
                s.tred.reattach(ti,'',END)  




    def sortColumn(s,col):

        global vts
       
        ic=listind(s.columns,col)        
        tis=s.tree.get_children()        
        dc=list(s.dti.items())
        vts=[(v,k) for k,v in dc if k in tis]

        ds=[]
        for(v,t) in vts:
            if v[-1]=='dr':
                try:
                    vl=v[ic].split('.')[0]
                    vl=int(vl)                        
                except:
                    vl=0                    
                ds.append([vl,t])

        fs=[]
        for(v,t) in vts:
            if v[-1]=='fl':
                try:
                    vl=v[ic].split('.')[0]
                    vl=int(vl)                        
                except:
                    vl=0                    
                fs.append([vl,t])          
           

        if ic==None:
            ic=0
        
        if ic==0:
            ds=[(v[ic].lower(),t) for (v,t) in vts if v[-1]=='dr']            
            fs=[(v[ic].lower(),t) for (v,t) in vts if v[-1]=='fl']
        else:
            ds=[(v[ic],t) for (v,t) in vts if v[-1]=='dr']            
            fs=[(v[ic],t) for (v,t) in vts if v[-1]=='fl']

        if s.rev==0:
            s.rev=1
            ds.sort()
            fs.sort()

        elif s.rev==1:
            s.rev=0
            ds.sort()
            fs.sort()
            ds=ds[::-1]
            fs=fs[::-1]

        df=ds+fs
        tis=[t for (v,t) in df]
            
        ax=[]
        for ti in tis:
            s.tree.reattach(ti,'',END)
            ax.append(s.dti[ti])
        s.datas[s.path]=ax

        tis=s.tree.get_children()
        s.files=[s.dtx[ti] for ti in tis]
        s.files=[e for e in s.files if isfile(e)]
        s.figs=filtrAny(s.files)
        s.orig=s.figs[:]

    def nearFigs(s,vl=1):        

        fgs=[]
        
        if len(s.fks)>0:
            fgs,ims=list(zip(*s.fks))
            s.acts=fgs[:]


        if s.lift=='canvas':

            if s.fig in s.acts:
                s.fig=near(s.acts,s.fig,vl)
            elif s.fig in s.figs:
                s.fig=near(s.figs,s.fig,vl)
            elif s.fig in s.rzn:
                s.fig=near(s.rzn,s.fig,vl)
            
            im=s.img

            fn=s.fig
            s.sve.set(s.fig)
            
            if isfile(s.fig):
                im=Image.open(s.fig)                

            elif fn.startswith('http'):
                im=urlImg(fn)
                
                if im:
                    s.toCanvas(fn,im)
                else:
                    s.nearFigs()                   
                

            elif fn in fgs:
                ic=fgs.index(fn)                
                fn,im=s.fks[ic]

            try:    
                s.toCanvas(fn,im)
            except Exception as ec:
                s.sta.config(text=ec)


        elif len(s.fks)>0:
            if vl==0:
                s.fk=choice(s.fks)
            else:
                try:
                    s.fk=near(s.fks,s.fk,vl)
                except:
                    s.fk=s.fks[0]
            fn,im=s.fk[:]
            s.toCanvas(fn,im)

        elif len(s.fis)>0:
            if vl==0:
                s.fk=choice(s.fis)
            else:
                try:
                    s.fk=near(s.fis,s.fk,vl)
                except:
                    s.fk=s.fks[0]
            fn,im=s.fk[:]
            s.toCanvas(fn,im)     

    def showFigs(s,vl):        

        def getZpFnIms(zn):
            
            s.zn=zn

            if zn.endswith('.zip'):                
                zf=ZipFile(zn)
            elif zn.endswith('.rar'):
                zf=RarFile(zn)
                
            ns=zf.namelist()
            js=filtrAny(ns,['.jpg'])
            if len(js)>0:
                fg=js[-1]
                try:
                    rd=zf.read(fg)                
                    ib=io.BytesIO(rd)
                    if ib:
                        im=Image.open(ib)
                        fnis.append((fg,im))
                        s.fgZp[fg]=zn
                except Exception as ex:
                    print(zn)
                    printex('getZpFnIms',ex)
                    s.sta.config(text=ex)
                    
            zf.close()

        def getFgIm(fg):

            if isfile(fg):

                try:                
                    im=Image.open(fg)
                    s.fis.append((fg,im))
                except:
                    s.nfi.append(fg)
                
            elif fg.startswith('http'):
                
                try:
                    ct=urlContent(fg)
                    ib=io.BytesIO(ct)
                    im=Image.open(ib)                                       
                    wd,he=im.size
                    if wd>350 and he>350:
                        s.fis.append((fg,im))

                except Exception as ex:
                    s.nfi.append(fg)
       
        s.frame.configure(background='SystemButtonFace')
        
        if vl==0:                
            
            if len(s.reFis)>0:
                lkv=len(s.dkv)
                s.preparFrame()
                s.stb.config(text='%d a %d de %d'%(s.ini,s.end,lkv))
                for fn,im in s.reFis:
                    s.addThumb(fn,im)
                s.hightlightScrFig()
                s.std.config(text='.dkv')
                return
                

            elif len(s.zipUdcRef)>0:
                
                lz=len(s.zipUdcRef)            
                s.preparFrame()            
                s.stb.config(text='%d a %d de %d'%(s.ini,s.end,lz))
                fnis=s.zipUdcRef[s.ini:s.end]
                fnis.sort()
                for fn,im in fnis:
                    s.addThumb(fn,im)
                s.hightlightScrFig()
                s.std.config(text='.zips')
                return               

            
        elif vl==1:
            s.ini+=s.qtd
            s.end+=s.qtd
            
        elif vl==-1:
            s.ini-=s.qtd
            s.end-=s.qtd
            
        if len(s.zpNs)>0:
            
            zns=s.zpNs[s.ini:s.end]
            fnis=[]
            concurrent(getZpFnIms,zns)
           
            s.preparFrame()
            for fg,im in fnis:
                s.addThumb(fg,im)                       

            s.fis=[]
            for (fn,im) in fnis:
                fg=nm2ur(fn)
                s.fis.append((fg,im))
                
            if len(fnis)>0:
                figs,ims=list(zip(*fnis))
                s.wTx.settext('\n'.join(figs))
                s.figs=figs
                s.zipUdcRef=fnis[:]

            s.stb.config(text='%d a %d de %d'%(s.ini,s.end,len(s.zpNs)))

        else:           
            fgs=s.figs[s.ini:s.end]            
            s.wBx.setlist(fgs)
            s.fis=[]
            s.nfi=[]
            concurrent(getFgIm,fgs)
            sleep(0.05)
            s.preparFrame()

            df=dict()
            for (k,v) in s.fis:
                df[k]=v            

            ax=[]

            for fg in fgs:
                if fg in df:
                    ax.append((fg,df[fg]))
            
            
            for (fn,im) in ax:
                s.addThumb(fn,im)
            s.refis=[]
            s.stb.config(text='%d a %d de %d'%(s.ini,s.end,len(s.figs)))

        try:
            cv=s.dfc[s.scrFig]
            cy=cv.winfo_y()
            fy=s.frame.winfo_height()
            alt=cy/fy
            s.wFr.yview('moveto',alt)            
            cv.configure(highlightbackground='blue')

        except Exception as ec:
            s.reFis=[]

    def hightlightScrFig(s):

        try:
            cv=s.dfc[s.scrFig]
            cy=cv.winfo_y()
            fy=s.frame.winfo_height()
            alt=cy/fy
            s.wFr.yview('moveto',alt)            
            cv.configure(highlightbackground='blue')

        except Exception as ec:
            pass
    

    def nearFiles(s,vl):

        if all([s.lift in ['frame','tred','canvas'], fileExt(s.path) in zipExt]):            

            ti=s.tree.focus()
            try:
                s.zip=s.dtx[ti]
            except:
                s.zip=s.path[:]
            vs=s.files            
           
            es=['.zip','.rar']
            
            s.zips=filtrEnd(vs,es)

            if len(s.zips)>0:
            
                if vl==0:
                    s.zip=choice(s.zips)
                    
                else:
                    
                    try:                
                        s.zip=near(s.zips,s.zip,vl)                    
                        
                    except:
                        s.zip=s.zips[0]

                    finally:
                        ti=s.dxt[s.zip]
                        s.send2filesHist(ti)
                        
                        s.tree.focus(ti)
                        s.tree.selection_clear()
                        s.tree.selection_set(ti)
                        s.tree.see(ti)                    
                
                s.mostrarZip(s.zip)

            else:
                showerror('Erro','Não há siblings')

        elif all([s.lift in ['frame','tred','canvas'],fileExt(s.path) in ['.txt',]]):
            
            tis=s.tred.get_children()
            ti=s.tred.focus()
            if not ti:
                ti=tis[0]
            
            nt=near(tis,ti,vl)
            s.tit.config(text=unquote(s.tred.item(nt,'values')[0]))
            s.tred.selection_clear()
            s.tred.selection_set(nt)
            s.tred.focus(nt)
            s.fromTred()            

        elif all([s.lift in ['frame','tred','canvas'],
                  fileExt(s.path) in ['.dkv','.uvs','.udc']]):   
            
            tis=s.tred.get_children()
            s.keys=[s.dtk[ti] for ti in tis]

            if not s.key:
                ti=s.tred.focus()
                try:
                    s.key=s.dtk[ti]
                except:
                    s.key=s.keys[0]
                
            s.key=near(s.keys,s.key,vl)
            
            s.tit.config(text=unquote(s.key))
            s.tit.update()
            
            fgs=s.dkv[s.key]
            
            s.figs=fgs
            
            s.stb.config(text='Figs: %d'%len(s.figs))
            s.orig=s.figs[:]
            s.figs2frame(fgs)

            ti=s.dkt[s.key]
            

            s.send2filesHist(ti)
           
            s.tred.selection_clear()
            s.tred.selection_set(ti)
            s.tred.focus(ti)
            s.tred.see(ti)

        elif all([s.lift in ['frame','tred','canvas'],'colado' in s.path]):
            s.showFigs(vl)

        elif isdir(s.path):
            s.showFigs(vl)
            
        elif s.lift=='text':

            fs=listfs(dirname(s.path))
            fs=filtrAny(fs,textExt)
            s.atx=near(fs,s.atx)
            s.mostrarTexto(s.atx)        
            
        else:
            pass

        ni=s.ini
        nd=s.end
        lf=len(s.figs)
        if nd>=lf:
            nd=lf
        ks=statusGet()
        if ks:
            s.stb.config(text='%d a %d de %d - %d ks'%(ni,nd,lf,ks))
        else:
            s.stb.config(text='%d a %d de %d'%(ni,nd,lf))

    def send2filesHist(s,ti):
        

        try:
            gt=s.dtx[ti]
            it=s.dti[ti]
            it[4]=agora()
            s.filesHist[gt]=it
            s.salvarConfigs()
        except Exception as exc:
            print(exc)

    def resetFilesHist(s):

        s.filesHist=dict()
        s.salvarConfigs()

    def fromLift(s):       

        if s.lift=='tree':
            s.fromTree()
            
        elif s.lift=='box':
            s.fromBox()
            
        elif s.lift=='tred':
            s.fromTred()
            
        elif s.lift=='text':
            s.fromText()        
            
    def fromTree(s):

        if s.dtx!={}:

            s.zipUdcRef=[]
            s.reFis=[]
            s.zpNs=[]
            s.fis=[]
            s.fks=[]

            ti=s.tree.focus()
            gt=s.dtx[ti]
            s.path=gt

            s.tit.config(text=unquote(s.path))

            s.ant='tree'

            if isdir(s.path):

                if s.path in s.datas:                    
                    s.data=tuple(s.datas[s.path])                    
                    s.makeTree()                    
                    
                else:                    
                    s.processData()

                s.dirsHist[s.path]=agora()
                s.salvarConfigs()
                
            elif isfile(gt):
                
                ti=s.tree.focus()
                s.send2filesHist(ti)
                s.mostrarArquivo(gt)
        
        s.zipUdcRef=[]

        s.scrFig=''
        s.scrRef=0
        s.reFis=[]

    def abrirArquivo(s):

        dr=''

        if isfile(s.path):
            dr=dirname(s.path)
        elif isdir(s.path):
            dr=s.path

        af=askopenfilename(initialdir=dr)
        if af:
            s.mostrarArquivo(af)

    def colarEndArquivo(s):

        gt=s.ms.clipboard_get()
        s.mostrarArquivo(gt)

    def mostrarArquivo(s,gt):        

        s.tit.config(text=gt)

        if endsAny(gt.lower(),imagExt):
            s.mostrarFileImage(gt)
    
        elif endsAny(gt.lower(),zipExt):#['.rar','.zip','.cbz','.cbr']):            
            s.mostrarZip(gt)

        elif endsAny(gt.lower(),dkvExt+['.uvs']):#['.udc','.dkv','.uvs']):                       
            s.mostrarDkv(gt)
        
        elif endsAny(gt.lower(),textExt):#['.txt','.py','.pyw','.srt','.wsp',
                                 #'.db','.js','.css','.htm','.html']):            
            s.mostrarTexto(gt)   

        elif endsAny(gt.lower(),dataExt):#['.csv','.json','.xlsx',
                                 #'.xltx','.xml','.xlsm','.pkl','.xls']):
            s.aqRead2Tred(gt)

        else:
            os.startfile(gt)

        tx='\n'.join(s.figs)
        s.wTx.settext(tx) 


    def seeParent(s):

        s.stop=1

        try:
            s.topProp.deiconify()
            
        except:
        
            if s.path=='':
                
                try:
                    s.path=list(s.datas.keys())[-1]
                except:
                    s.path=''       

            if all([s.lift in ['frame','text'],
                    s.ant in ['tred','canvas']]):
                s.liftWidget('tred')
                s.sta.config(text='')
                ks=statusGet()
                s.stb.config(text='%d Ks'%ks)
                return


            s.liftWidget('tree')
            s.tree.yview('moveto',0)
            
            cpd=copy(s.path)         
            s.path=dirname(s.path)
            
            if s.path in s.datas:                
                s.data=tuple(s.datas[s.path])                
                s.makeTree()
                ti=s.tree.focus()
                s.tree.see(ti)
                s.tit.config(text=unquote(s.path))               

            else:
                
                s.path=dirname(s.path)
                s.scanPath()

            if cpd!=s.path:
                
                try:
                    ti=s.dxt[cpd]
                    s.tree.selection_set(ti)                    
                    s.tree.focus(ti)
                    s.tree.focus_set()
                    s.tree.see(ti)                    
                except:
                    pass

            try:
                ti=s.tree.focus()
                s.tree.see(ti)
            except:
                pass
            
            

            s.dds=[e for e in s.data if e[-1]=='dr']
            s.dfs=[e for e in s.data if e[-1]=='fl']

            tis=s.tree.get_children()                
                
            dr=s.path
            ld=len(s.dds)
            lf=len(s.dfs)
            sz=sum([s.dti[ti][3] for ti in tis])
            fz=formatedSize(sz)
                 
            tx='Ds: %d - Fs: %d - Sz: %s'%(ld,lf,fz)
            s.sta.config(text='')
            s.tit.config(text=unquote(s.path))
            s.stb.config(text=tx)
            s.stc.config(text='Lift: Tree')

    def verNoExplorer(s):

        if isfile(s.path):
            os.startfile(dirname(s.path))
            
        elif isdir(s.path):
            os.startfile(s.path)

    def verNoBrowser(s):        

        if s.lift=='tree':
            
            ti=s.tree.focus()
            fl=s.tree.item(ti)['values'][0]
            fl=rpath(pjoin(s.path,fl))
            for et in ['.txt','.dkv','.zip']:
                fl=fl.replace(et,'')
            s.toHtml(fl)            

        elif s.lift=='text':
            
            ur=s.getTextLine()
            s.toHtml(ur)

        elif s.lift=='box':

            fn=s.box.get(ACTIVE)

            if 'http' in fn:
                ur=fn
                s.toHtml(ur)
                
            else:           
            
                s.rzf.extract(fn,'temp')
                cd=os.getcwd()
                ur='%s/temp/%s'%(cd,fn)
                ur=rpath(ur)
                s.toHtml(ur)
            
        elif s.lift=='frame':

            s.textName2Web()

        elif s.lift=='canvas':          

            if isfile(s.fig):
                s.toHtml(s.fig)

            elif isUi(s.fig):
                os.startfile(s.fig)

            elif s.fig in s.textlines:
                os.startfile(s.fig)
                
            elif s.fig in s.rzn:
                fn=s.fig                
                s.rzf.extract(fn,'temp')
                cd=os.getcwd()
                ur='%s/temp/%s'%(cd,fn)
                ur=rpath(ur)
                s.toHtml(ur)                

            elif s.fig in s.fgZp:

                fg=s.fig   
                zn=s.fgZp[fg]
                zf=ZipFile(zn)
                zf.extract(fg,'temp')
                cd=os.getcwd()
                ur='%s/temp/%s'%(cd,fg)
                ur=rpath(ur)
                s.toHtml(ur)

            else:
                s.browseMaxDims()

            
        elif s.lift=='tred':
            
            ti=s.tred.focus()
            ur=s.dtd[ti]        
        
            bn=basename(ur)
            ur=pjoin(s.path,bn)
            s.toHtml(ur)

        elif ur.startswith('http'):
            os.startfile(ur)

        elif s.rzf!=None  and ur in s.rzf.namelist():    
            
            if isUi(nm2ur(ur)):
                os.startfile(nm2ur(ur))
            else:            
                s.toHtml(ur)            

    def toHtml(s,fl=None):

        if fl==None:
            fl=s.fig

        bn=basename(fl)
        ur=nm2ur(bn)
        
        fl=quote(fl)            
        fil="file:///%s"%fl            
        alt='\n'.join(wrap(fl,30))           
       
        with open('tempHt.html','w') as ht:
            w=ht.write
            w('<!DOCTYPE html >')            
            w('<html target="_self">')           
            w('<body>')                
            w('<center><img height=400px src={}></center>'.format(fil))
            w('</body></html>')
            os.startfile('tempHt.html')

    def getPjoin(s):

        ts=s.tree.focus()
        ti=s.tree.index(ts)        
        bn=s.data[ti][0]
        dn=s.data[ti][-2]        
        return pjoin(dn,bn)

    def liftWidget(s,wt):
        
        s.entryOut()
        
        if wt=='tree':
            s.ftr.lift()
            s.ms['menu']=s.Men
            try:
                s.verTreeFocus()
            except:
                pass                    
           
        elif wt=='canvas':
            s.wCv.lift()
        
            
        elif wt=='box':
            s.wBx.lift()
            s.ms['menu']=s.Men
            
        elif wt=='text':
            
            s.wTx.lift()
            s.ms['menu']=s.Men                       
            
            try:
                s.tex.focus()
                s.tex.tag_add('sel',s.sel_firt,s.sel_last)
                s.tex.see(s.sel_first)
            except:
                try:
                    if s.fig in s.figs:
                        s.tex.selection_clear()
                        s.destacar(s.fig)
                except:
                    pass
            
            s.ms['menu']=s.Men
            ll=len(s.textlines)
            lf=len(s.figs)

        elif wt=='frame':
            s.ms['menu']=s.Men
            s.wFr.lift()            
            s.stc.config(text='Lift: Frame')
           
        elif wt=='tred':
            
            s.frTd.lift()                 
           
               
        s.lift=wt
        s.stc.config(text='Lift: %s'%wt)        

        #lifts

        if s.lift in ['tree','tred','box']:
            s.ms.unbind('<Up>')
            s.ms.unbind('<Down>')
            s.ms.bind('<Prior>',Fc(s.scrollPage,-1))
            s.ms.bind('<Next>',Fc(s.scrollPage,1))            
            s.ms.bind('<Control-z>',Fc(s.seeParent))
            s.ms.bind('<Control-Z>',Fc(s.seeParent))
            s.ms.bind('<Delete>',Fc(s.removerDoDisco))
            s.ms.bind('<Right>',Fc(s.nearFiles,1))
            s.ms.bind('<Left>',Fc(s.nearFiles,-1))

            
        elif s.lift in ['frame']:
            s.ms.bind('<Up>',Fc(s.scrollPage,-1))
            s.ms.bind('<Down>',Fc(s.scrollPage,1))
            s.ms.bind('<Control-z>',Fc(s.seeParent))
            s.ms.bind('<Control-Z>',Fc(s.seeParent))
            s.ms.bind('<Delete>',Fc(s.removerDoDisco))

        elif s.lift in ['canvas']:
            s.ms.bind('<Right>',Fc(s.nearFigs,1))
            s.ms.bind('<Left>',Fc(s.nearFigs,-1))


        elif s.lift in ['text',]:
            s.ms.bind('<Prior>',Fc(s.scrollPage,-1))
            s.ms.bind('<Next>',Fc(s.scrollPage,1))
            s.ms.unbind('<Control-z>')
            s.ms.unbind('<Control-Z>')
            s.ms.unbind('<Delete>')
            s.ms.unbind('<Right>')
            s.ms.unbind('<Left>')

        if isfile(s.path):
            fil,ext=splitext(s.path)
            s.std.config(text='Ext: %s'%ext)
        else:
            s.std.config(text='')
            
    def destacar(s,gt): 
        
        if gt:
            ind='1.0'
            ind=s.tex.search(gt,
                             ind,
                             nocase=1,
                             )
            lastind='%s+%dc'%(ind,len(gt))
            
            if ind and lastind:
                s.tex.tag_remove('found', '1.0', END)                
                s.tex.tag_add('found',ind,lastind)
                s.tex.tag_config('found',foreground='red')
                s.tex.see(ind)
        

    def mostrarZip(s,gt):
        
        s.zip=gt 
        
        def notRoz(ex):
            s.tit.config(fg='red')
            tx='%s\n%s'%(s.zip,ex)
            s.tit.config(text=tx)
            s.clearFrame()
            return

        if s.zip:         

            s.lift='frame'
            s.data=tuple([])
            s.fns=[]
            s.dtr=dict()
            s.path=s.zip

            if splitext(s.zip)[1] in ['.zip','.cbz','.ZIP','.CBZ']:
                try:
                    s.rzf=ZipFile(s.zip)
                except Exception as ex:
                    notRoz(ex)

            elif splitext(s.zip)[1] in ['.rar','.cbr','.RAR','.CBR']:            
                try:
                    s.rzf=RarFile(s.zip)
                except Exception as ex:
                    notRoz(ex)            
            
            s.tit.config(fg='blue')                
            s.rzl=s.rzf.infolist()
            s.rzn=s.rzf.namelist()
            
            s.rzn=s.rzn[::-1]
            s.stb.config(text='%d figs'%len(filtrAny(s.rzn)))
            
            s.orig=s.rzl[:]
            s.toFrameTred() 
        
            
    def toFrameTred(s):

        s.preparFrame()
        
        s.makeTred()
        
        s.yview=s.tred.yview
        s.wBx.setlist([])
        
        s.fis=[]
        s.stop=0
        
        for fn in s.rzn:
            if s.stop==1:
                break
                
            try:
                
                if containAny(fn,imagExt):                    
                    rd=s.rzf.read(fn)
                    ib=io.BytesIO(rd)        
                    im=Image.open(ib)
                    s.fis.append((fn,im))                    
                    
            except Exception as ex:
                print('toFrameTred')
                print(e)
                print(ex)
                print()

        s.figs=filtrAny(s.rzn)
        s.wBx.setlist(s.figs)

        s.orig=s.figs[:]
        s.oths=list(set(s.rzn)-set(s.figs))
        
        lf=len(s.figs)
        lo=len(s.oths)         
            
        if lf>=lo:
            s.liftWidget('frame')
            
        elif lf<lo:
            s.liftWidget('tred')
                
        s.dfc=dict()
        s.dcf=dict()
        
        for (fn,im) in s.fis:
            s.addThumb(fn,im)            
        s.stc.config(text='Lift: Frame')
        s.fks=s.fis[:]
        

    def sair(s):

        try:
            s.salvarConfigs()
        except:
            pass
       
        s.ms.destroy()           

    def preparFrame(s):

        chs=s.frame.winfo_children()
        for ch in chs:
            ch.destroy()        
        s.frame.focus_force()
        
        s.liftWidget('frame')
        s.ms.update()
        
        s.wid=s.canvas.winfo_width()
        hei=s.canvas.winfo_height()

        wd=s.wid
        ld=s.ldo+4

        s.row=0
        s.col=-1

        s.lim=int((wd)/(ld))       
        s.lcv=int((wd-22)-4*s.lim)/s.lim

    def addThumb(s,fn,im):
        
        w,h=im.size

        l1=int(s.lcv)
        l2=h*l1/w
        s.fn=fn
        
        def cavHolder():
            
            cv=Canvas(s.frame,
                      width=l1,
                      height=l2,
                      background='SystemButtonFace',
                      highlightbackground='White',
                      )
            
            cv.update()            
            
            s.col+=1
            if s.col>=s.lim:
                s.col=0
                s.row+=1
            cv.grid(row=s.row,column=s.col)

            if im!=None:                
                cp=copy(im)
                cp.thumbnail(size=(l1,l2))
                s.ph=ImageTk.PhotoImage(cp)  
                x,y=wigCenter(cv)                
                cv.create_image(x,y,image=s.ph)
                cv.image=s.ph
                
                s.dfc[fn]=cv
                s.dcf[cv]=fn
                
            else:
                
                np=npartit(fn)
                nm='\n'.join(np)
                
                x,y=wigCenter(cv)
                s.canvas.create_text(x,y,nm)
            
            cv.bind('<1>',Fc(s.preAmp,fn,im))
            cv.bind('<2>',Fc(s.renewThumb,cv,cp))
            cv.bind('<Enter>',Fc(s.statShow,fn,im))

        try:
            cavHolder()

        except:
            try:
                cavHolder()
            except:
                pass

    def statShow(s,fn,im):

        s.fn=fn

        bn=basename(fn)
        wd,he=im.size
       
        di=list(s.dfc.items())
        ns,cs=zip(*di)

        ic=ns.index(fn)+1

        ni=s.ini
        nd=s.end

        lf=len(s.figs)

        ni=s.ini
        nd=s.end
        if nd>=lf:
            nd=lf  

        if splitext(s.path)[1]=='.dkv':            

            if lf>0:                
                s.dvk=dkv2dvk(s.dkv)

                try:
                    ky=s.fgKv[fn]                    
                    lk=len(s.dkv[ky])                    
                    tx='%s - %d figs'%(ky,lk)                                       

                except:                     
                    tx='%s - %d figs'%(fn,len(s.fis))
                    
                tx=unquote(tx)
                s.sta.config(text=unquote(tx[-272:]))

        s.sta.config(text=unquote(fn[-272:]))
        
        if fileExt(s.path)=='':
            s.stb.config(text='%d a %d de %d'%(s.ini,s.end,len(s.zpNs)))
            
        elif fileExt(s.path) in dkvExt:
            s.stb.config(text='%d a %d de %d'%(s.ini,s.end,len(s.dkv)))

        elif isdir(s.path):
            s.stb.config(text='%d a %d de %d'%(s.ini,s.end,len(s.figs)))
            
        s.stc.config(text='%d x %d'%(wd,he))
        
        if isfile(s.path):
            tx=fileExt(s.path)
            s.std.config(text=tx)                

    def renewThumb(s,cv,im):

        l2,l2=wigCenter(cv)
        s.ph=ImageTk.PhotoImage(im)
        cv.create_image(l2,l2,image=s.ph)
        cv.image=s.ph        

    def preAmp(s,fn,im):

        s.ant='frame'
        
        if fn in s.fgPt:
            s.path=s.fgPt[fn]            
            
        s.toCanvas(fn,im)
        
        try:
            ti=s.ddt[fn]
            s.tred.selection_clear()
            s.tred.selection_set(ti)
            s.tred.see(ti)
        except:
            pass

    def togWidget(s):

        wgs=s.widgets
        ic=wgs.index(s.lift)
        ic+=1
        ic=ic%len(wgs)
        s.liftWidget(wgs[ic])

    def mostrarDkv(s,fl):

        s.zipUdcRef=[]
        s.zpNs=[]

        s.path=fl
        src=fl

        dc={}
            
        if fl.endswith('.uvs'):
            dc=uvs2dkv(fl)                    
                
        elif s.path.endswith(('.dkv','.udc')):
            dc=openDkv(fl)            

        s.dkv=dc
        s.dkv2tred()
        

        vls=list(s.dkv.values())
        
        lks=len(s.dkv)        
        lvs=sum([len(vs) for vs in vls])
        
        s.stb.config(text='Keys: %d - Figs: %d'%(lks,lvs))
        s.path=fl

        s.figs=[vs[0] for vs in vls if len(vs)>0]        
       

    def udcFigs(s):

        s.ini,s.end=0,50
        s.qtd=50

        if s.lift in ['tree','tred']:
            s.mostrarUdc()

        elif s.lift=='canvas':
            s.udc2frame()


    def mostrarUdc(s):        
        
        
        s.ref=s.path[:]
        s.zipUdcRef=[]

        s.scrFig=''
        s.scrRef=0
        s.reFis=[]

        if all([isfile(s.path),fileExt(s.path) in dkvExt+['.uvs']]):            

            s.fgKv=dict()
            s.ini,s.end=0,s.qtd
            s.figs=[]

            tis=s.tred.get_children()
            keys=[s.tred.item(ti,'values')[1] for ti in tis]            
            
            for ky in keys:
                
                vs=s.dkv[ky]
                vs=filtrEnd(vs)
                if len(vs)>0:
                    ch=vs[-1]
                    s.figs.append(ch)
                    s.fgKv[ch]=ky
            
            tx='\n'.join(s.figs)
            s.wTx.settext(tx)
            s.showFigs(0)            
            s.zipUdcRef=[]
            s.zpNs=[]

        elif isdir(s.path):  

            tis=s.tree.get_children()            
            ff=[s.dtx[ti] for ti in tis]
            kys=filtrEnd(ff,zipExt)
            s.zpNs=filtrEnd(kys,zipExt)
            
            if len(s.zpNs)>0:
                s.showFigs(0)
            else:
                pass

        s.tit.config(text=unquote(s.path))
        s.std.config(text='UDC')
        
    def udc2frame(s):

        s.scrFig=s.fig[:]        
        s.frame.configure(background='goldenrod2')        
        
        def getFgIm(fg):            
                
            try:
                ct=urlContent(fg)
                ib=io.BytesIO(ct)
                im=Image.open(ib)                                       
                wd,he=im.size
                if wd>350 and he>350:
                    s.fks.append((fg,im))
            except:
                pass

        fg=s.fig        
       
        if fg in s.fgKv:            

            ky=s.fgKv[fg]            
            fs=s.dkv[ky]
            s.fks=[]
            concurrent(getFgIm,fs)

            s.preparFrame()

            for (fg,im) in s.fks:
                s.addThumb(fg,im)

            s.reFis=s.fis[:]
            s.std.config(text='.dkv')

        elif fg in s.fgZp:           

            zn=s.fgZp[fg]           
            s.zip=zn            
            s.mostrarZip(s.zip)

            s.sta.config(text=unquote(fg))
            s.stc.config(text=zn)

            tis=s.tree.get_children()
            s.std.config(text='.zips')
            try:
                ti=s.dxt[zn]
                s.tree.selection_clear()
                s.tree.selection_set(ti)
                s.tree.focus(ti)
                s.tree.see(ti)
            except:
                pass

        else:
           
            s.liftAnterior()
            s.hightlightScrFig()
             
       

    def uvs2dkv(s):        

        op=open(s.path)
        rd=op.read()
        op.close()

        sp=rd.split('\n\n')
        dc=dict()
             
        def adc(et):
            li=et.split('\n')
            try:
                ky=li[0]
                vs=li[1:]
                dc[ky]=vs
            except:
                pass

        np=npartit(sp)
        ln=len(np)
        
        for ic,pt in enumerate(np):            
            concurrent(adc,pt)        

        s.dkv=dc
        s.dkv2tred()

    def text2dkv(s):

        if s.lift=='text':

            kvFp='temp/temp.dkv'
            newDkv(kvFp)

            tx=s.tex.get('0.0',END)
            li=tx.split('\n')
            dc=keysValues(li)
            
            saveDkv(dc,kvFp)
            s.dkv=openDkv(kvFp)
            s.dkv2tred()

            s.path=kvFp

        elif s.lift=='box':

            kvFp='temp/temp.dkv'
            newDkv(kvFp)

            li=s.box.get(0,END)            
            dc=keysValues(li)
            
            saveDkv(dc,kvFp)
            s.dkv=openDkv(kvFp)
            s.dkv2tred()
            s.path=kvFp            

    def text2kvExt(s):

        if s.lift=='tred':
            tis=s.tred.get_children()
            fgs=[s.tred.item(ch,'values')[0] for ch in tis]
            s.dkv=dc=kvExt(fgs)

        elif s.lift=='text':

            tx=s.tex.get('0.0',END)
            li=tx.split('\n')
            s.dkv=dc=kvExt(li)

        s.dkv2tred()

    def dkv2tred(s):

        s.liftWidget('tred')
        s.makeTred()        
        s.stb.config(text='lines: %d'%len(s.dtd))
        s.stc.config(text='lift: %s enc: %s'%(s.lift,s.enc))

    def setEncoding(s,cd):

        s.enc=cd
        fl=s.path

        if fl.endswith(('.wsp','.db','.pkl')):
            
            tl=textli(fl)
            np=npartit(tl)
            for pt in np:
                tx='\n'.join(pt)
                s.tex.insert(END,tx)
                
        if cd=='get enc':
            
            enc=getEncoding(s.path)
            s.enc=enc
            
            op=open(s.path,'r',encoding=enc)
            rd=op.read()
            op.close()
            
            s.wTx.settext(rd)
            tl=rd.split('\n')
            
        else:
            
            tl=textLines(fl)
            tx='\n'.join(tl)
            s.wTx.settext(tx)
            
        s.textlines=tl    
        
        tt=len(tl)
        s.figs=filtrAny(tl)
        s.orig=s.figs[:]
                
        s.lift='text'                
        s.tit.config(text=unquote(s.path))
        
        s.stb.config(text='%d lines'%tt)
        s.stc.config(text='lift: %s enc: %s'%(s.lift,s.enc))

       
    def mostrarTexto(s,fl):         

        s.atx=fl       
        
        s.path=fl        
        s.textlines=[]
        
        s.liftWidget('text')
        s.ms.update()

        s.wTx.configure(text_foreground='black')

        if fl.endswith(('.wsp','.db','.pkl')):
            
            tl=textli(fl)
            tl=flat(tl)
            np=npartit(tl)
            for pt in np:
                tx='\n'.join(pt)
                s.tex.insert(END,'%s\n'%tx)           
            
        else:
            s.stb.config(text='INICIANDO')
            s.wTx.configure(text_foreground='black')            
            
            try:
                enc=getEncoding(fl)                               
            except:
                enc='utf-8'           
                
            op=open(fl,'r',encoding=enc)     
            tl=[]
            
            nr=0
            
            while 1:
                
                try:
                    le=op.readline()
                    if le=='':
                        break
                    le=unquote(le.rstrip())
                    
                    if le.count('http')>1:
                        le=le[le.rfind('http'):] 
                    tl.append(le)                       
                        
                except:
                    continue
        
            
            
            s.ms.update()
                
            s.textlines=tl  
            
            tt=len(tl)
            s.figs=filtrAny(tl)            
            s.orig=s.figs[:]
                    
            s.lift='text'
            s.wTx.configure(text_foreground='purple')
            s.tit.config(text=unquote(s.path))
            
            s.stb.config(text='%d lines'%tt)
            s.stc.config(text='lift: %s enc: %s'%(s.lift,s.enc))

        s.figs=tl
        tx='\n'.join(tl)
        s.wTx.settext(tx)
       
            
    def singleLines(s):

        if s.lift=='text':

            s.textlines=singli(s.textlines)
            rd='\n'.join(s.textlines)
            s.wTx.settext(rd)
            s.liftWidget('text')
            
            s.figs=filtrAny(s.textlines,imagExt)
            s.orig=s.figs[:]
                
            if len(s.figs)>0:
                s.fig=s.figs[0]
            s.ant='text'
            
            s.stb.config(text='%d lines'%len(s.textlines))

    def mostrarFileImage(s,fl):
        
        s.fig=fl        
        
        try:
            im=Image.open(fl)
            s.toCanvas(fl,im)
            s.ant='tree'
            s.arq=fl           
            
        except Exception as ec:            
            
            cv=s.canvas
            cv.delete(ALL)
            x,y=wigCenter(cv)
            
            st=str(ec)
            wp=wrap(st,width=60)            
            jo='\n'.join(wp)
            tx='%s\n\n%s'%(fl,jo) 
            
            cv.create_text(x,y,
                           text=tx,
                           fill='red')

    def fromBox(s):

        gt=s.box.get(ACTIVE)
        
        s.tit.config(text=unquote(s.path))

        if isfile(gt):
            s.mostrarArquivo(gt)            

        elif isdir(gt):

            if gt in s.datas:
                
                s.data=tuple(s.datas[gt])
                s.makeTree()
                s.liftWidget('tree')

            else:
                s.path=gt
                s.scanPath()
                s.addDir(gt) 

        elif gt in s.dkv:
            fgs=s.dkv[gt]
            s.toFrameTred()
            s.urlsHist[gt]=agora()

        elif gt in s.rzn:
            fn=gt
            s.rzf.extract(fn,'temp')
            cd=os.getcwd()
            ur='%s/temp/%s'%(cd,fn)
            ur=rpath(ur)
            s.toHtml(ur)

        elif gt.startswith('http'):
            webbrowser.open(gt)
            s.urlsHist[gt]=agora()

        s.path=gt
        s.tit.config(text=unquote(s.path))

    def fromRoz(s,fn):

        try:
            fl=s.rzf.extract(fn,'temp')
            os.startfile(fl)            
        except:            
            os.startfile(fn)

        try:
            s.salvarConfigs()            
        except Exception as ex:
            print('fromRoz')
            print(ex)
            print()

    def getTextLine(s):

        try:
            gi=s.tex.index(INSERT)        
            lin,col=gi.split('.')        

            le,cl=int(lin),int(col)     

            i1='%d.0'%le
            i2='%d.end'%le        

            line=s.tex.get(i1,i2)
            line=line.split()[0].strip(punct)
           
            if 'http' in line:
                line=line[line.rindex('http'):].replace("'",'')            
                
            return line
        
        except:
            return

    def curTextLine(s):

        i=s.tex.index(INSERT).split('.')[0]        
        line=s.tex.get('%s.0'%i,'%s.end'%i)
        return line.strip()  

    def fromText(s):

        line=s.curTextLine()        

        if all(e in line for e in list('[-]')):
            dd=desdob(line)
            s.figs2frame(dd)

        elif isfile(line):
            im=Image.open(line)
            s.toCanvas(line,im)
            s.ant='text'
            return

        elif 'http' in line:
            if isUi(line):
                im=urlImg(line)
                if im:
                    s.toCanvas(line,im)
                else:
                    s.nearFigs()
                s.ant='text'
                return
            else:
                webbrowser.open(line)
           
        elif line in s.dkv:            
            vs=s.dkv[line]
            s.key=line
            s.produzirHtml(vs)
            
            if len(vs)<=1: 
                os.startfile(line)

        elif all(e in line for e in list('[-]')):
            dd=desdob(line)
            s.produzirHtml(dd)

        elif 'file:///' in line:
            line=unquote(line.replace('file:///',''))           
            os.startfile(line)
       

        s.lift='text'
        tt=len(s.textlines)
        s.wTx.configure(text_foreground='purple')
        s.tit.config(text=unquote(s.path))
        
        s.stb.config(text='%d lines'%tt)
        s.stc.config(text='lift: %s enc: %s'%(s.lift,s.enc))

        try:
            s.salvarConfigs()
        except Exception as ex:
            print('fromText')
            print(ex)
            print()

    def verFavoritos(s):

        s.wBx.setlist(s.favoritos)

##        dats=[('Nr','Urls','Data')]
##        di=list(s.favoritHist.items())
##        di=[(k,v) for (k,v) in di if isdir(k)]
##        s.favoritsHist=dict(di)
##        s.salvarConfigs()
##        di=[(k,invDat(v)) for (k,v) in di]
##        di=di[::-1]
##
##        for ic,(ur,dt) in enumerate(di):
##            vs=(ic,ur,dt)
##            dats.append(vs)
##
##        s.path=''
##        s.lift='tred'
##        
##        s.configTred(dats)
##        s.histConfs()
##        
##        s.tit.config(text='Histórico de favoritos')
##        s.sta.config(text='Histórico de favoritos')
##        s.stb.config(text=len(di))        

        s.liftWidget('box')
        s.wBx.setlist(s.favoritos)
        
    def verFilesHist(s):
        
        s.clearTree()
        
        its=list(s.filesHist.items())
        its=[(ky,vl) for (ky,vl) in its if isfile(ky)]

        if len(its)>0:
            fs,acs=list(zip(*its))
          
            s.wBx.setlist(fs)        

            s.dtx=dict()        
            s.dxt=dict()
            s.dti=dict()

            for fl,it in its:            
                    
                bn,ex,fs,sz,mt,bs,dn,tg=it
                if fileExt(fl) in zipExt:                
                    zf=ZipFile(fl)
                    fs=len(zf.infolist())-1
                    zf.close()
                    
                fz=fmSize(sz)
                dt=invDat(mt)
                vs=bn,ex,fs,fz,dt,bs
                ti=s.tree.insert('',END,values=vs,tag=tg)            
                    
                s.dtx[ti]=fl            
                s.dxt[fl]=ti
                s.dti[ti]=it

            s.liftWidget('tree')


            s.tit.config(text='Files Hist')
            s.sta.config(text='Files Hist')
            s.stb.config(text=len(its))
            s.stc.config(text='')
            s.std.config(text='')
            s.tree.tag_configure('fl',foreground='purple')

            s.filesHist=dict(its)
            s.salvarConfigs()

    def verUrlsHist(s):
    
        dats=[('Nr','Urls','Data')]
        di=list(s.urlsHist.items())
        di=[(k,v) for (k,v) in di if isUrl(k)]
        s.urlsHist=dict(di)
        s.salvarConfigs()
        di=[(k,invDat(v)) for (k,v) in di]
        di=di[::-1]

        for ic,(ur,dt) in enumerate(di):
            vs=(ic,ur,dt)
            dats.append(vs)

        s.path=''
        s.lift='tred'
        
        s.configTred(dats)
        s.histConfs()
        
        s.tit.config(text='Histórico de urls')
        s.sta.config(text='Histórico de urls')
        s.stb.config(text=len(di))        

    def verExpandHist(s):

        dats=[('Nr','Urls','Data')]
        di=list(s.expandHist.items())
        di=[(k,v) for (k,v) in di if isUrl(k)]
        s.expandHist=dict(di)
        s.salvarConfigs()        
        di=[(k,invDat(v)) for (k,v) in di]
        di=di[::-1]

        for ic,(ur,dt) in enumerate(di):
            vs=(ic,ur,dt)
            dats.append(vs)

        s.path=''
        s.lift='tred'           
       
        s.configTred(dats)
        s.histConfs()
        
        s.tit.config(text='Histórico de expandidas')
        s.sta.config(text='Histórico de expandidas')
        s.stb.config(text=len(di))
        s.stc.config(text='')
        s.std.config(text=s.lift)                     
        
    def verDirsHist(s):

        dats=[('Nr','Dirs','Data')]
        di=list(s.dirsHist.items())
        di=[(k,v) for (k,v) in di if isdir(k)]
        s.dirsHist=dict(di)
        s.salvarConfigs()
        di=[(k,invDat(v)) for (k,v) in di]
        di=di[::-1]

        for ic,(ur,dt) in enumerate(di):
            vs=(ic,ur,dt)
            dats.append(vs)

        s.path=''
        s.lift='tred' 

        s.configTred(dats)
        s.histConfs()

        s.lift='tred'
        s.tit.config(text='Histórico de diretórios')
        s.sta.config(text='Histórico de diretórios')
        s.stb.config(text=len(di))
        s.stc.config(text='')
        s.std.config(text=s.lift)


    def liftAnterior(s):

        s.liftWidget(s.ant)
        s.ms.title('ttkFolderSizes')
        s.endSlider()

    def toCanvas(s,fn,im):

        s.fig=fn.strip()        
        s.img=copy(im)

        if isfile(fn):
            
            bn=basename(fn)            
            if 'http%3A' in bn:
                ur=n2u(bn)
            else:
                ur=nm2ur(bn)

        s.arq=fn

        s.ent.xview(END)        
        s.canvas.update()        
        cw,ch=wigSize(s.canvas)
        iw,ih=im.size

        s.iw,s.ih=iw,ih
        sz=iw,ih
        
        if s.rsz!='None':            

            if s.rsz=='Width':
                sz=fitWidth(cw,ch,iw,ih)
                
            elif s.rsz=='Height':
                sz=fitHeight(cw,ch,iw,ih)
               
            im=im.resize(sz)            
            iw,ih=im.size
            
        s.ph=ImageTk.PhotoImage(im)     
        s.canvas.update()
        
        x=max((cw,iw))
        y=max((ch,ih))

        s.canvas.delete(ALL)
        s.liftWidget('canvas')
        
        s.canvas.create_image(x/2,y/2,
                              image=s.ph,
                              tag='img')
        s.canvas.image=s.ph        
        
        s.wCv.resizescrollregion()
        s.ent.config(fg='blue')

        s.fgPt[s.fig]=s.path

        #####
##        ext=fileExt(s.path)
##
##        if ext in zipExt:
##            s.fgZp[s.fig]=s.zip
##        elif ext in dkvExt:
##            s.fgKv[s.fig]=s.path
        #####
        
        fg=s.fig
        tm=time()
        s.fgTmIm[fg]=(tm,im)
        s.salvarConfigs()
       
        try:
            ti=s.dxt[fn]
            s.tree.selection_set(ti)
            
            s.tree.focus(ti)
            s.tree.focus_set()
            
            s.tree.see(ti)
        except:
            pass       

        try:            
            ky=s.dvk[s.fig]
            s.key=ky
            ti=s.dkt[ky]            
            s.tred.selection_set(ti)
            s.tred.focus(ti)
            s.tred.focus_set()
            s.tred.see(ti)
        except:
            pass
        
        s.lift='canvas'
        
        wd,he=im.size
        tx='%d x %d'%(wd,he)
        s.sta.config(text=unquote(s.fig))
        s.stb.config(text=tx)
        s.stc.config(text='Lift: Canvas')
        s.ms.title('ttkFolderSize - %s - %d x %d'%(fn,wd,he))
        s.sve.set(nm2ur(fn))
        s.ent.xview(END)

    def mouseScroll(s,ev):

        dt=ev.delta
       
        if dt>0:
            vl=-2
        else:
            vl=2

        li=[('canvas',s.wCv),
            ('tree'  ,s.tree),
            ('box'   ,s.wBx),
            ('text'  ,s.wTx),
            ('frame' ,s.wFr),
            ('tred'  ,s.tred),]        
        
        for k,v in li:
            if s.lift==k:
                wg=v
                
        wg.yview('scroll',vl,'units')

    def setFilt(s,ft):
        
        s.filt=ft

    def userprofile(s):

        s.path=os.environ['USERPROFILE']
        s.scanPath()

    def tamanhoDoLado(s):

        ai=askinteger('Lado','Entre com o tamanho\ndo lado (nr. inteiro)',
                      initialvalue=s.ldo)
        if ai:
            s.ldo=ai
            s.thumbnails()

    def getSel(s):        

        if all([s.lift=='tred',s.tredCols==['Nr','Urls','Qtds']]):
            ti=s.tred.focus()
            ky=s.dtk[ti]
            sl=s.dkv[ky]
            s.key=ky                       
        
        elif s.lift=='text':

            try:
                i1=s.tex.index(SEL_FIRST).split('.')[0]
                i2=s.tex.index(SEL_LAST).split('.')[0]
                i1='%s.end'%i1
                i2='%s.end'%i2
                gt=s.tex.get(i1,i2)
                sp=gt.split('\n')
                
                s.sel_first=i1
                s.sel_last=i2
                sl=[et.replace(' ','') for et in sp]
                
            except:
                gt=s.tex.get('0.0',END)
                sp=gt.split('\n')
                sl=[et.replace(' ','') for et in sp]
           
        elif all([s.lift=='tree',splitext(s.path)[1].lower() not in '.zip .rar'.split()]):

            sel=s.tree.selection()            
            
            if len(sel)==0:
                return(s.figs)
            
            elif len(sel)==1:
                ti=sel[0]                
                fn=s.tree.item(ti)['values'][0]
                qt=quote(fn)
                fp=pjoin(s.path,qt)
                
                if fp.endswith(('.zip','.rar')):
                    zf=ZipFile(fp)
                    nl=filtrAny(zf.namelist())
                    sl=[nm2ur(et) for et in nl]
                    
                elif fp.endswith(('.txt','.dkv','.udc')):
                    dc=openDkv(fp)
                    sl=list(dc.values())
                    
                else:
                    sl=s.figs                    

            else:
                sl=[s.dtx[ti] for ti in sel]
                sl=filtrAny(sl)

        elif s.lift=='tred' and splitext(s.path)[1] in dataExt:

            tis=s.tred.selection()            
            if len(tis)==0:                
                tis=s.tred.get_children()
                
            t1=tis[1]            
            le=s.tred.item(t1).get('values')            
            le=[str(e) for e in le]            
            iu=listind(le,'http')            
            sl=[s.tred.item(ti,'values')[iu] for ti in tis]

        elif all([s.lift in ['tred','frame','box'],
                  splitext(s.path)[1] in ['.zip','.rar']]):

            s.rzf.extractall('temp')
            cd=os.getcwd()
            sl=['%s/temp/%s'%(cd,e) for e in s.rzn]
            sl=[rpath(fg) for fg in sl]

        elif all([s.lift in ['tred','frame'],
                  splitext(s.path)[1] in ['.dkv','.udc','txt']]):

            try:
                ti=s.tred.focus()
                ky=s.dtk[ti]
                sl=s.dkv[ky]
                s.key=sl
            except:
                sl=filtrAny(s.textlines)
            
        elif s.lift=='box':
            sl=s.box.curselection()
            if len(sl)==0:
                sl=s.box.get(0,END)
                sl=s.figs


        elif all([s.lift=='tree',isdir(s.path)]):
                    
            sel=s.tree.selection()
            ls=len(sel)

            if ls>0:
                ls=sel
            
            elif len(sel)==0:
                fls=list(s.dtx.values())
                sl=filtrAny(fls)

        elif all([s.lift=='frame',isdir(s.path)]):

            fgs=s.dcf.values()
            sl=[unquote(fg) for fg in fgs]            
            
        elif s.lift=='canvas':
            sl=[s.fig,]

        s.sel=sl
        return sl

    def thumbnails(s,sl=None):

        if len(s.figs)>0:

            s.ini,s.end=0,50

            ak=askstring('Interv','%d a %d de %d'%(s.ini,s.end,len(s.figs)),
                         initialvalue='%s %s'%(s.ini,s.end))
            
            if ak:
                sp=ak.split()
                s.ini,s.end=int(sp[0]),int(sp[1])
           
            s.showFigs(0)
        
       
    def produzirHtml(s,sl=None,ur=None):
               
        if sl==None:
            sl=s.getSel()
            sl=singli(sl)

        if len(sl)==1:
            s.verNoBrowser()       
        
        elif len(sl)>1:
            
            fh=filtrAny(sl,['http'])
            if len(fh)>0:
                s.makeHtml(fh)
            else:
                try:
                    sl=[nm2ur(e) for e in sl]        
                    s.makeHtml(sl)
                except:
                    sl=[nm2ur(basename(e)) for e in sl]
                    s.makeHtml(sl)
            
    def makeHtml(s,sl=None,ur=None):      
        
        s.ms.update()
        sw=s.ms.winfo_screenwidth()
        wd=(sw/5)-40

        np=npartit(sl,5)
        
        with open('html.html','w') as ht:
            w=ht.write
            w('<!DOCTYPE html>\n')            
            w('<html><body>\n')
            
            if ur!=None:
                ur=unidecode(ur)
                w('<a id="ur" href="{}"><center>{}</center></a>'.format(ur,ur))
                
            w('<table>\n')
            s.stop=0
            for pt in np:
                if s.stop==1:
                    break
                w('<tr>\n')
                for fg in pt:
                    try:
                        nr=sl.index(fg)
                    except:
                        nr=''                        
                    tf=len(sl)
                    
                    try:
                        wp=unidecode('\n'.join(wrap(fg,25)))                    
                        w('<td>\n')
                        w('''<a href="{}"><img src="{}" width="{}px"
                             title="{} de {}" figs alt="{}">\n
                             '''.format( fg,fg,wd,nr,tf,wp))
                        
                    except:
                        continue
                w('</td>\n')    
                w('</tr>\n')            
                w('</table\n>')            
                w('</body></html>\n')
                
        webbrowser.open('html.html')

    def setResize(s,sz):

        s.rsz=sz
        try:            
            s.toCanvas(s.fig,s.img)
        except:
            pass

    def changeResize(s):        

        rz=['Width','Height','None']
        s.rsz=near(rz,s.rsz,1)        
        s.toCanvas(s.fig,s.img)

    def abrirNoExplorer(s):

        os.startfile(s.path)

    def setWrap(s,wr):

        s.tex.configure(wrap=wr)

    def findInFiles(s):

        def pesquisar():

            s.stop=1
            s.ms.update()

            s.liftWidget('text')
            s.wTx.settext('')

            ge=s.sve.get()
            gs=ge.split()            

            tis=s.tree.get_children()
            ln=len(tis)            

            vist=[]
            s.stop=0
            s.fa=[]

            s.stb.config(fg='red')

            s.stop=0
            for ic,ti in enumerate(tis):
                if s.stop==1:
                    break
                try:
                    tf=s.dtx[ti]
                    tl=textli(tf)
                    fa=filtrAny(tl,gs)
                    s.fa.extend(fa)
                        
                    for et in s.fa:
                        if not et in vist:                        
                            
                            s.tex.insert(END,'%s\n'%et)
                            vist.append(et)
                            try:
                                ds.write('%s\n'%et)
                            except:
                                pass
                           
                    s.tex.see('end')
                    s.stb.config(text='%d/%d'%(ic+1,ln))

                    s.ms.update()

                    s.tex.yview('moveto',0)
                    s.stc.config(text=len(vist),fg='black')
                    
                    s.textlines=vist

                except Exception as ex:
                    print('findInFiles')
                    print(ex)
                    print()

        ay=askyesno('Pesquisa',
                    'Quer salvar a pesquisa?')

        if ay:
            nm=s.sve.get()+'.txt'
            sv=asksaveasfilename(initialdir='',
                                 defaultextension='.txt',
                                 initialfile=nm)
            s.dst=sv
            if sv:
                s.arq=sv
                with open(sv,'w') as ds:
                    pesquisar()
            else:
                pass
        else:
            pesquisar()


    def zippar(s):        

        if s.lift=='text':
            
            gl=s.getTextLine()
            
            if gl in s.dkv:
                fs=s.dkv[gl]
                s.key=gl
            else:
                fs=s.getSel()       
            
            fo=fs[0]                        
            fl=ur2nm(fo)+'.zip'
            s.makeZip(fl,fs)            

        elif all([s.lift in ['box'], len(s.figs)>0]):

            fl=s.tit['text']
            fl=ur2nm(fl)+'.zip'
            s.makeZip(fl,s.figs)        

        elif all([s.lift in ['frame','canvas'], len(s.fis)>0]):

##            if not fileExt(s.path) in zipExt:

            if s.url!='':
                fl=ur2nm(s.url)+'.zip'
            else:
                fl=ur2nm(s.fis[0][0])+'.zip'
            fgs,ims=list(zip(*s.fis))
            s.makeZip(fl,fgs)

##            else:
##                showinfo('Info','"%s"\nis a zipfile already'%s.path)

        elif all([s.lift in ['tred','frame'], len(s.tredCols)==3]):

            try:
            
                ti=s.tred.selection()[0]
                ky=s.dtk[ti]
                fs=s.dkv[ky]
                fl=ur2nm(ky)+'.zip'
                s.makeZip(fl,fs)

            except:
                
                fl=ur2nm(s.fig)+'.zip'
                fs=list(zip(*s.fks))[0]
                s.makeZip(fl,fs)
            

        elif s.lift=='tree':

            tis=s.tree.selection()
            fs=[pjoin(s.path,s.tree.item(ti,'values')[0]) for ti in tis]
            fl=fs[0]+'.zip'
            s.makeZip(fl,fs)            
              

    def makeZip(s,fl,fs):

        ft=([('Zips',('*.zip','*.rar')),('Todos','*.*')])
        zn=asksaveasfilename(initialdir='T:/bunny',
                             initialfile=fl,
                             defaultextension='.zip',
                             filetypes=ft)
        s.ms.update()
        
        if zn:
            bn=basename(zn)
            dn=dirname(zn)
            lf=len(fs)            
            ak=askokcancel('Zippar',
'''
Será criado o arquivo:

    Zip:
%s

    No diretório:
%s

    Com
%d figuras
    
'''%(bn,dn,lf))
            if ak:
                tf=open('transfs.db','wb')                
                dump((zn,fs),tf)
                tf.close()
                os.startfile('transf.pyw')


    #--------------------------------------------
        

    def salvarAlteracoes(s):

        if s.lift=='text':
            s.salvarTexto()
            tl=textLines(s.path)
            s.stb.config(text=len(tl))

      
        elif s.lift=='tred':
            ak=askokcancel('Salvar','Tem certeza que quer \nsalvar alterações em \n"%s"'%s.path)
            if ak:
                if s.path in dataExt:
                    s.dst=s.path[:]
                    s.saveFromTred()
  

    def salvarTextoComo(s):

        sv=asksaveasfilename()
        if sv:
            s.path=sv
            s.salvarTexto()

    def salvarTexto(s):

        gt=s.tex.get('0.0',END)
        gs=gt.split('\n') 

        if all([s.lift=='text',fileExt(s.path) in textExt]):
            with open(s.path,'w') as aq:
                for le in gs:
                    le=unidecode(le)
                    aq.write('%s\n'%le)

        s.sta.config(text='Salvo em %s'%unquote(s.path[-72:]))
        s.mostrarTexto(s.path)

    def salvarDkv(s):

        fn,ex=splitext(s.path)
        bn=basename(fn)
        fl='%s.dkv'%bn

        sv=asksaveasfilename(initialdir='D:/PyDevelop3/webSpy/DKVs',
                             initialfile=fl,
                             defaultextension='.dkv',
                             )
        s.ms.update()
        if sv:
            saveDkv(s.dkv,sv)
            s.sta.config(text=unquote(sv[-72:]))
            s.mostrarDkv(sv)            

    def mostrarDkvsMaiores(s):
            
        di=list(s.dkv.items())        
        mais=[(ky,filtrAny(vs,['.jpg'])) for(ky,vs) in di]
        mais=[(ky,vs) for (ky,vs) in mais if len(vs)>=5]
        
        s.dkv=dict()
        for (ky,vs) in mais:
            s.dkv[ky]=vs
            s.key=ky
               
        s.dkv2tred()

    def textName2Web(s):

        if s.lift=='tree':
            ti=s.tree.focus()
            fn=s.tree.item(ti)['values'][0]
            bn=basename(fn)
            
            ur=nm2ur(bn)
            ur=ur[ur.rfind('http'):]
            for et in['.txt','.dkv','.zip']:
                ur=ur.replace(et,'')

        elif s.lift=='text':

            ur=s.getTextLine()

        elif s.lift=='box':
            gt=s.box.get(ACTIVE)
            if urOk(gt):
                ur=gt
            else:
                ur=nm2ur(gt)

        elif s.lift in ['tred']:
            
            ti=s.tred.focus()
            vs=s.tred.item(ti,'values')
            ic=listind(vs,'http')
            ur=fn=s.tred.item(ti)['values'][ic]

        elif s.lift in['frame']:

            if fileExt(s.path) in zipExt:
                zp=s.zip
                ur=nm2ur(zp)
                try:
                    ic=ur.rindex('http')
                    ur=ur[ic:]
                except:
                    pass

            elif fileExt(s.path) in dkvExt:
                kv=s.dkv
                ur=nm2ur(kv)            
           
        elif s.lift=='canvas':
            if s.fig in s.rzn:
                zp=s.zip
                ur=nm2ur(zp)
            elif s.fig.startswith('http'):
                ur=nm2ur(dirname(s.fig))

        if urOk(ur):
            os.startfile(ur)
            
        else:
            try:
                ur=dirname(s.figs[0])
                os.startfile(ur)
            except:
                showinfo('Info','No such url:\n%s'%ur)
            

    def recursivo(s):

        s.aqs=0
        s.ttt=0

        if s.path!='':

            ff=folderFiles(s.path)
            s.path=pjoin(s.path,' - recursivo')
            
            np=npartit(ff)
            s.delTree()

            s.figs=[]
            s.orig=s.figs[:]

            s.stop=0
            for pt in np:
                if s.stop==1:
                    break
                s.flVals=[]
                concurrent(s.addFile,pt)

                for fl,item,vals in s.flVals:
                    ti=s.tree.insert('','end',values=vals,tags='fl')
                
                    s.dtx[ti]=fl
                    s.dti[ti]=item
                    s.dxt[fl]=ti
                    s.tree.yview('moveto',1)

                    s.figs.append(fl)
                
                s.tree.tag_configure('fl',foreground='black')
                s.ms.update()
            
            tis=s.tree.get_children()    
            s.files=[s.dtx[ti] for ti in tis]

            s.figs=filtrAny(s.files)
            uris=[nm2ur(et) for et in s.figs]
            text='\n'.join(uris)
            s.wTx.settext(text)
            
            s.tree.tag_configure('dr',foreground='blue',)
            s.tree.tag_configure('fl',foreground='purple')
            s.tree.yview('moveto',0)
            s.stb.config(text='%d arquivos'%len(s.figs))

    def criarRenomeador(s):
        
        s.tpr=Toplevel()
        s.tpr.withdraw()
        s.tpr.protocol('WM_DELETE_WINDOW',s.tpr.withdraw)
        
        s.lbRenam=Label(s.tpr,
                    text='',
                    padx=10,
                    pady=5)
        s.lbRenam.bind('<1>',s.lbRenam2cb)
        
        s.renam=StringVar()
        en=Entry(s.tpr,
                 font=font1,
                 fg='blue',
                 textvariable=s.renam,
                 )
        en.bind('<Return>',Fc(s.reName))
        
        fi=Frame(s.tpr)
        fi.grid(row=2,column=1)
        
        bt=Button(fi,text='ok',width=6,command=Fc(s.reName))
        cc=Button(fi,text='Cancel',width=6,command=Fc(s.cancelRename))        
        
        s.lbRenam.grid(row=0,column=1)
        grid1(en)
        
        bt.grid(row=0,column=1,padx=1,pady=5)
        cc.grid(row=0,column=2,padx=1)
        
        centralizar(s.tpr)
        en.focus()

    def cancelRename(s):

        s.renam.set('')
        s.tpr.withdraw()
        
    def lbRenam2cb(s,ev):

        s.ms.clipboard_clear()
        tx=s.lbRenam['text']
        s.ms.clipboard_append(tx)
        s.renam.set(tx)

    def renomear(s): 
        
        s.tpr.deiconify()

        if s.lift=='canvas':
            bn=basename(s.fig)            
            s.lbRenam.configure(text=bn)            
            
        elif s.lift=='tree':
            ti=s.tree.focus()            
            bn=s.tree.item(ti,'values')[0]            
            s.lbRenam.configure(text=bn)

        elif s.lift=='tred':
            ti=s.tred.focus()            
            it=s.tred.item(ti,'values')[0]
            bn=basename(it)
            s.lbRenam.configure(text=bn)              

    def reName(s):       

        if s.lift=='tree':
            
            '''
            s.dtx[ti]=dr
            s.dti[ti]=item
            s.dxt[dr]=ti
            '''                       
                       
            ti=s.tree.focus()            
            vs=list(s.dti[ti])
            
            od=vs[0]
            nw=s.renam.get() 
            vs[0]=nw            
            
            s.tree.item(ti,values=vs)
            s.tpr.withdraw()
            
            old=s.path
            par=dirname(s.path)
            new=pjoin(par,nw)
            os.rename(old,new)

            s.dtx[ti]=nw
            s.dti[ti]=vs
            s.dxt[nw]=ti 
            
            s.datas[s.path]=tuple(s.dti.values())
            s.salvarConfigs()            

        elif s.lift=='tred':
            pass

        elif s.lift=='canvas':
            pass
        

    def toTrash(s,wt):

        os.chmod(wt,stat.S_IWRITE)
        
        try:
            
            shutil.move(wt,'trash',)
            
        except Exception as ex:
            print('toTrash')
            print(fl)
            print(ex)
            print()
        

    def makeTred(s):

        s.tit.config(text=unquote(s.path))

        if s.path.endswith(('.zip','.rar')):
            

            if s.rzf!=None:            

                s.clearTred()
                s.zipConfs()
                
                s.dtd=dict()
                s.ddt=dict()
                s.dtf=dict()                

                figs=[]
                s.stop=0
                
                for e in s.rzf.infolist():

                    if s.stop==1:
                        break

                    try:
                        rd=s.rzf.read(e)
                        ib=io.BytesIO(rd)        
                        im=Image.open(ib)
                        wd,he=im.size
                        
                    except:
                        wd,he=0,0
                        
                    fn=e.filename
                    figs.append(fn)
                    ex=splitext(fn)[1]                
                    sz=e.file_size
                    dt=e.date_time
                    fz=formatedSize(sz)

                    a,m,d,H,M,S=dt[:6]
                    mt='%.2d/%.2d/%.4d %.2d:%.2d:%.2d'%(d,m,a,H,M,S)

                    item=[fn,ex,wd,he,sz,dt]
                    vals=[fn,ex,wd,he,fz,mt]
                    
                    ti=s.tred.insert('','end',values=vals,tag='le')
                    
                    s.dtd[ti]=item 
                    s.dtf[ti]=fn
                    s.ddt[fn]=ti                    

        elif s.path.endswith(('.dkv','.udc','.uvs','.txt')):
            
            if s.path.endswith('.uvs'):
                s.dkv=uvs2dkv(s.path)

            s.clearTred()
            s.dkvConfs()
                
            di=s.dkv.items()
            
            s.dtk=dict()
            s.dtd=dict()
            s.dkt=dict()
            
            for ic,(ky,vs) in enumerate(di):
                
                vals=(ic,ky,len(vs))
                ti=s.tred.insert('','end',values=vals,tag='le')
                
                s.dtk[ti]=ky
                s.dtd[ti]=vals
                s.dkt[ky]=ti

               
        tis=s.tred.get_children()
        fgs=[s.tred.item(ch,'values')[0] for ch in tis]

        tx='\n'.join(fgs)
        s.wTx.settext(tx)
        s.orig=s.figs[:]
        s.tred.tag_configure('le',foreground='purple')
        tx='Keys: %d - Figs: %d'%(len(tis),len(s.dcf))
        s.stb.config(text=tx)
        s.wBx.setlist(s.figs)
        
        
    def clearTred(s):

        tis=s.tred.get_children()
        for ch in tis:
            s.tred.detach(ch)


    def sortTred(s,col):        
        
        for cl in s.tredCols:            
            if col.lower() in cl.lower():
                ic=listind(s.tredCols,cl)
                if ic:
                    break       
        
        tis=s.tred.get_children()
        vals=[(s.dtd[ti],ti) for ti in tis]
        lvt=['time','data',]              
        
        if containAny(col.lower(),lvt):            
            et=filtrAny(s.tredCols,lvt)            
            ic=s.tredCols.index(et[0])            
            dt=[(invDat(vl[ic]),ti) for (vl,ti) in vals]
        else:
            dt=[(vl[ic],ti) for (vl,ti) in vals]

        try:    
            dt.sort(reverse=s.rev)
        except:
            dt=[(str(et),ti) for (et,ti) in dt]
            dt.sort(reverse=s.rev) 
        
        if s.rev==False:
            s.rev=True
            
        elif s.rev==True:
            s.rev=False       

        ax=dict()
        for ic,it in enumerate(dt):
            vl,ti=it
            s.tred.reattach(ti,'',END)
            ax[ti]=s.dtd[ti]
            
        s.dtd=ax

                
    def fromTred(s):

        global sti,fn
       
        ti=s.tred.focus()        
        s.ant='tred'
        sti=s.tred.item(ti,'values')
        et1=sti[1]
               
        
        try:
            ic=listind(sti,'http')
            fn=sti[ic]
            if all(e in fn for e in list('[-]')):
                dd=desdob(fn)
                s.ms.clipboard_clear()
                
                tx='\n'.join(dd)
                s.ms.clipboard_append(tx)
                s.proExpand()
            else:                        
                os.startfile(fn)
            
        except:
            
            ic=1
            fn=sti[1]
            
        if fn in s.rzn:
            s.rzf.extract(fn,'temp')
            fl='%s/temp/%s'%(os.getcwd(),fn)
            os.startfile(fl)

        elif isfile(fn):
            try:
                s.mostrarArquivo(fn)
            except:
                os.startfile(fn)


        elif isdir(et1):
            s.path=et1
            
            if et1 in s.datas:                
                s.data=tuple(s.datas[s.path])                    
                s.makeTree()                
            else:                
                s.processData()
            return         

##        elif all(['http' in fn,fileExt(fn) in imagExt]):            
##            
##            try:
##                ct=urlContent(fn)
##                ib=io.BytesIO(ct)
##                im=Image.open(ib)
##                s.toCanvas(fn,im)
##            except Exception as ec:
##                s.sta.config(text=ec,foreground='red')                
    
        elif isdir(s.path):

            fn=s.tred.item(ti,'values')[1]
            
            if all([s.rzf!=None, fn in list(s.rzn)]):

                rd=s.rzf.read(fn)
                ib=io.BytesIO(rd)
                
                try:
                    im=Image.open(ib)
                    s.img=im
                    s.ant='tred'
                    s.toCanvas(fn,im)
                    
                except:
                    cd=os.getcwd()
                    s.rzf.extract(fn,'temp')
                    fl='%s/%s/%s'%(cd,'temp',fn)
                    if fl.endswith('.txt'):
                        op=open(fl)
                        rd=op.read()
                        op.close()
                        s.wTx.settext(rd)
                        s.wTx.lift()
                    else:
                        os.startfile(fl)

                s.path=s.zip
            

        elif s.path.endswith(('.txt','.py','.pyw','.srt','.wsp','.db',
                              '.pkl','.js', '.css','.htm','.html')):
            s.mostrarTexto(s.path)        

        elif endsAny(s.path.lower(),dataExt):
            
            li=list(s.tred.item(ti,'values'))            
            ic=listind(li,'http') 
            ur=s.tred.item(ti,'values')[ic]                
            os.startfile(ur)            
   
        elif endsAny(s.path.lower(),zipExt):
            
            ti=s.tred.focus()
            fn=s.tred.item(ti,'values')[1]
            
            try:
                fgs,ims=zip(*s.fis)
                if fn in fgs:
                    ic=fgs.index(fn)
                    fg,im=s.fis[ic]
                    s.toCanvas(fg,im)

            except:        
                zf=ZipFile(s.path)
                ns=zf.namelist()
                if fn in ns:
                    zf.extract(fn,'temp')
                    fl='%s/temp/%s'%(os.getcwd(),fn)
                    os.startfile(fl)                    

        elif splitext(s.path)[1] in dataExt:
            
            ti=s.tred.focus()            
            zp=zip(s.columns,s.dtd[ti])
            s.showMask(zp)

        else:
            vs=s.tred.item(ti,'values')
            ts=[str(et) for et in vs]
            try:
                ic=listind(ts,'http')
            except:
                ic=1
            ky=ts[ic]

            '''
            s.dtf[ti]=fn
            s.dtd[ti]=item
            s.dft[fn]=ti
            '''        

            ky=unquote(ky)   
            s.tit.config(text=ky)
            s.ini,s.end=0,50

            if all([s.tredCols==[u'Nr',u'Urls',u'Qtds'],
                    s.path.endswith(('.dkv','.udc','.uvs','.txt'))]):           
             
                s.key=ky=s.dtk[ti]            
                s.figs=s.dkv[ky]
                s.stb.config(text='Figs: %d'%len(s.figs))
                s.orig=s.figs[:]           
               
                if len(s.figs)<=1:                
                    os.startfile(ky)
                else:                
                    s.figs2frame(s.figs)

                s.tred.focus(ti)
                s.tred.selection_clear()
                s.tred.selection_set(ti)
                s.tred.see(ti)        

    def figs2frame(s,fgs=None):        
        
        s.dtd=dict()
        s.dfi=dict()

        s.wFr.focus_set()

        s.nfi=[]

        ni=s.ini
        nd=s.end
        
        lf=len(s.figs)
        if nd>lf:
            nd=lf

        tx=basename(s.path)[-72:]
        tx=unquote(tx)
        
        s.sta.config(text=tx)
        s.stb.config(text='%d a %d de %d'%(ni,nd,lf))
        s.stc.config(text=s.lift)
        s.std.config(text='')

        def getFgIm(fg):

            if isfile(fg):

                try:                
                    im=Image.open(fg)
                    s.fis.append((fg,im))
                except:
                    s.nfi.append(fg)
                
            elif fg.startswith('http'):
                
                try:
                    ct=urlContent(fg)
                    ib=io.BytesIO(ct)
                    im=Image.open(ib)                                       
                    wd,he=im.size
                    if wd>350 and he>350:
                        s.fis.append((fg,im))

                except Exception as ex:
                    s.nfi.append(fg)

            elif fg in s.rzn:

                try:
                
                    rd=s.rzf.read(fg)
                    ib=io.BytesIO(rd)        
                    im=Image.open(ib)
                    wd,he=im.size
                    if wd>350 and he>350:
                        fg=nm2ur(fg)
                        s.fis.append((fg,im))
                except:
                    s.nfi.append(fg)

        s.fis=[]
        fgs=filtrAny(fgs)
        concurrent(getFgIm,fgs)
                
        s.dfc=dict()
        s.dcf=dict()
        s.dtd=dict()

        s.preparFrame()
        
        if len(s.fis)>0:
            
            for (fg,im) in s.fis:
                s.addThumb(fg,im)

            dc={'tree'  :s.tree,
                'tred'  :s.tred,
                'frame' :s.wFr,
                'text'  :s.wTx,
                'canvas':s.wCv,
                'box'   :s.wBx,}
            
            wg=dc[s.lift]
            wg.yview('moveto',0)

        s.nfi.insert(0,'não convertidas')
        s.fks=s.fis[:]
        s.wBx.setlist(fgs)

    def scrollPage(s,vl):
        
        vl*=15     

        dc={'tree'  :s.tree,
            'tred'  :s.tred,
            'frame' :s.wFr,
            'text'  :s.wTx,
            'canvas':s.wCv,}

        wt=dc[s.lift]        
        wt.yview('scroll',vl,'units')
    
    def processTranspose(s):        
           
        w,h=s.img.size
        l1=int(s.lcv)
        l2=h*l1/w

        cv=s.dfc[s.fig]
        l1=int(s.lcv)
        l2=h*l1/w
        
        cp=copy(s.img)        
        cp.thumbnail(size=(l1,l2))
        s.ph=ImageTk.PhotoImage(cp)  
        x,y=wigCenter(cv)                
        cv.create_image(x,y,image=s.ph)
        cv.image=s.ph
        cv.update()
        
        if isfile(s.fig):
            s.img.save(s.fig)
            s.img=Image.open(s.fig)
            s.toCanvas(s.fig,s.img)

    def flipHorizontal(s):
        
        s.img=s.img.transpose(Image.Transpose.FLIP_LEFT_RIGHT)
        s.toCanvas(s.fig,s.img)
        
        if isfile(s.fig):
            s.processTranspose()
            s.img.save(s.fig)
            s.img=Image.open(s.fig)            
            s.updateCanvasThumb()
        
    def flipVertical(s):

        s.img=s.img.transpose(Image.Transpose.FLIP_TOP_BOTTOM)
        s.toCanvas(s.fig,s.img)
        
        if isfile(s.fig):
            s.processTranspose()
            s.img.save(s.fig)
            s.img=Image.open(s.fig)
            s.toCanvas(s.fig,s.img)
            s.updateCanvasThumb()

    def rotacionarImagem(s):

        ai=askinteger('rotacionarImagem',
                      'Entre com o valor',
                      initialvalue=90)
        if ai:
            
            s.img=s.img.rotate(-ai,expand=True)
            s.toCanvas(s.fig,s.img)
            
            if isfile(s.fig):
                s.img.save(s.fig)
                s.img=Image.open(s.fig)
                s.updateCanvasThumb()            
            
    def updateCanvasThumb(s):

        try:
            cv=s.dfc[s.fig]            
            cp=copy(s.img)
            w,h=s.img.size
            l1=int(s.lcv)
            l2=h*l1/w
            cp.thumbnail(size=(l1,l2))
            s.ph=ImageTk.PhotoImage(cp)  
            x,y=wigCenter(cv)                
            cv.create_image(x,y,image=s.ph)
            cv.image=s.ph
            s.dfc[s.fig]=cv
            
        except Exception as ex:
            print('updateCanvasThumb')
            print('rotacionar')
            print(s.fig)
            print(ex)
            print()

    def mouseRoll(s,ev):
        
        dt=ev.delta
       
        if dt>0:
            vl=1
        elif dt<0:
            vl=-1

        s.nearFigs(vl)

    def fromDatas(s):

        if s.path in s.datas:
            s.data=tuple(s.datas[s.path])
            s.makeTree()

            try:
                s.salvarConfigs()
            except Exception as ex:
                print('fromDatas')
                print(ex)
                print()

    def updateDatas(s): 

        ls=list('DEFGHIJKLMNOPQRSTUVWXYZ')
        ds=['%s:/'%lt for lt in ls]

    def unquoteText(s):

        if s.lift=='text':

            gl=getLines(s.path)
            nw=[unquote(le.replace('file:///','')) for le in gl]
            tx='\n'.join(nw)
            s.wTx.settext(tx)

        elif s.lift=='box':

            gl=s.box.get(0,END)
            nw=[unquote(le) for le in gl]
            s.wBx.setlist(nw)

    def names2urls(s):

        if s.lift=='text':

            gl=getLines(s.path)
            nw=[nm2ur(et) for et in gl]
            tx='\n'.join(nw)
            s.wTx.settext(tx)

        elif s.lift=='box':

            gl=s.box.get(0,END)
            nw=[nm2ur(et) for et in gl]
            s.wBx.setlist(nw)

        elif s.lift=='tree':

            tis=s.tree.get_children()
            gl=[s.tree.item(ti,'values')[0] for ti in tis]
            nw=[nm2ur(et) for et in gl]
            s.wBx.setlist(nw)
            s.liftWidget('box')

        elif s.lift=='tred':

            tis=s.tred.get_children()
            gl=[s.tred.item(ti,'values')[1] for ti in tis]
            nw=[nm2ur(et) for et in gl]
            s.wBx.setlist(nw)
            s.liftWidget('box')

    def repriseData(s):

        try:
            s.data=tuple(s.datas[s.path])
            
            try:
                s.makeTree()
            except:
                s.makeTred()
                
        except:
            s.scanPath()

    def criarScTexTopLev(s):

        s.tlv=Toplevel()
        s.tlv.protocol('WM_DELETE_WINDOW',Fc(s.tlv.withdraw))
        s.tlv.withdraw()
        s.tti=Label(s.tlv)
        s.tti.grid(row=0,column=1)
        s.ttx=scrolledtext.ScrolledText(s.tlv,
                         font=font1,
                         fg='blue',
                         width=5,
                         height=15,
                         padx=20,
                         )
        
        grid1(s.ttx)
        s.ttx.bind('<Double-1>',Fc(s.gotoLine))
        s.tst=Label(s.tlv)
        s.tst.grid(row=2,column=1)

    def findInTex(s):
        
        s.tex.tag_remove('found', '1.0', END)
        gt=askstring('Find','Find what?')
        if gt:            

            cs=[]
            ind='1.0'
            
            if gt:
                
                while True:
                    ind=s.tex.search(gt,
                                     ind,
                                     nocase=1,
                                     stopindex=END,
                                     )
                    if not ind:
                        break
                    
                    cs.append(ind)
                    lastind='%s+%dc'%(ind,len(gt))
                    s.tex.tag_add('found',ind,lastind)
                    
                    ind=lastind                    
                    s.tex.tag_config('found',foreground='red')
                    
                s.mostrarTexIcs(gt,cs)

    def mostrarTexIcs(s,gt,cs):

        s.tlv.deiconify()
        s.ttx.delete(1.0,END)
        for ic in cs:
            s.ttx.insert(END,'%s\n'%ic)
            
        s.tti['text']=gt
        s.tst['text']=len(cs)                
        
        centralizar(s.tlv)
        s.tlv.update()
        s.tlv.lift()

    def gotoLine(s):

        at=s.ttx.index(INSERT)        
        ln,cl=at.split('.')        
        tx=s.ttx.get('%s.0'%ln,'%s.end'%ln)
        
        tx=tx.strip()              
        s.tex.see(tx)

    def showTexIndic(s):

        ic=s.tex.index(INSERT)
        s.stc.config(text='lift: %s enc: %s ic: %s'%(s.lift,s.enc,ic))

    def groupDirs(s):

        li=textLines(s.path)
        li=[e for e in li if e!='']
        s.wTx.settext('')

        s.dkv=groupDirs(li)
        s.dkv2tred()

    def zipConfs(s):

        s.tredCols=(
               u'Arquivos',
               u'Ext',                   
               u'Width',
               u'Height',
               u'Tamanho',
               u'Data',
               )

        s.tred.configure(columns=s.tredCols)

        for col in s.tredCols:
            s.tred.heading(col,
                           text=col,
                           anchor=CENTER,
                           command=Fc(s.sortTred,col),
                           )

        cfs=[             
             ('#1', 400, W),
             ('#2',  60, E),
             ('#3',  50, E),
             ('#4',  50, E),
             ('#5',  80, E),
             ('#6', 160, CENTER),
             ]
        
        for nr,wd,ac in cfs:
            t=s.tred.column(nr,
                            width=wd,
                            anchor=ac,
                            stretch=NO,
                            )

    def dkvConfs(s):

        s.tredCols=[u'Nr',u'Urls',u'Qtds']
        s.tred.configure(columns=s.tredCols)
        ancs=[E,CENTER,E]

        colAnc=zip(s.tredCols,ancs)

        for col,anc in colAnc:
            s.tred.heading(col,
                           text=col,
                           anchor=anc,
                           command=Fc(s.sortTred,col),
                           )

        cfs=[             
             ('#1',  40,E),
             ('#2', 600,W),
             ('#3',  60,E)
             ]
        
        for nr,wd,ac in cfs:
            t=s.tred.column(nr,
                            width=wd,
                            anchor=ac,
                            stretch=NO,
                            )

    def histConfs(s):

        s.tredCols=[u'Nr',u'Urls',u'Data']
        s.tred.configure(columns=s.tredCols)
        ancs=[E,W,CENTER]

        colAnc=zip(s.tredCols,ancs)

        for col,anc in colAnc:
            s.tred.heading(col,
                           text=col,
                           anchor=anc,
                           command=Fc(s.sortTred,col),
                           )

        cfs=[             
             ('#1',  40,E),
             ('#2', 600,W),
             ('#3', 150,CENTER)
             ]
        
        for nr,wd,ac in cfs:
            t=s.tred.column(nr,
                            width=wd,
                            anchor=ac,
                            stretch=NO,
                            )


    def gotoHome(s):

        dc={'tree'  :s.tree,
            'tred'  :s.tred,
            'frame' :s.wFr,
            'text'  :s.wTx,
            'canvas':s.wCv,}
        
        wg=dc[s.lift]
        wg.yview('moveto',0)  

    def gotoEnd(s):

        dc={'tree'  :s.tree,
            'tred'  :s.tred,
            'frame' :s.wFr,
            'text'  :s.wTx,
            'canvas':s.wCv,}
        
        wg=dc[s.lift]
        wg.yview('moveto',1)  
      
    def aqRead2Tred(s,fp=None):
      
        dats=s.dataRead(fp)

        s.configTred(dats)
        s.textlines=dats[:]        

    def dataRead(s,fp):       
      
        s.ms.update()
        nam,ext=splitext(fp)
        ext=ext.lower()

        if ext=='.csv':            
            df=pd.read_csv(fp)
           
        elif ext=='.json':            
            df=pd.read_json(fp)
            
        elif ext in ['.xlsx', '.xls' ,'.xltx','.xlsm']:
            df=pd.read_excel(fp)
            
        elif ext=='.xml':
            df=pd.read_xml(fp)

        elif ext=='.pkl':
            df=pd.read_pickle(fp)

        try:
            cs=df.columns.to_list()
            
        except Exception as ex:
            print('dataRead')
            print(fp)
            print(ex)
            print()        
        
        
        try:
            s.pdf=df
        except:
            pass
            
        cls=tuple(df.columns)
        ds=[tuple(e) for e in df.values]
        dats=[cls]+ds

        return dats    

    def configTred(s,dats):

        s.utd=0
        s.dcv=dcColVals(dats)

        s.anc=W
        s.pdf=pd.DataFrame(s.dcv)
        s.ms.update()
        
        s.tredCols=list(dats[0])        
        dats=dats[1:]        
        
        s.tred.configure(columns=s.tredCols)
        
        for col in s.tredCols:            
            s.tred.heading(col,
                           text=col,
                           command=Fc(s.sortTred,col),
                           anchor=s.anc,
                           )
            
        mwd=s.ms.winfo_width()
        
        lcl=len(s.tredCols)
        
        if lcl>0:
            wd=int(mwd/lcl)
        else:
            wd=int(mwd/3)
        
        cfs=[('#%d'%(ic+1),wd) for (ic,et) in enumerate(s.tredCols)]
        s.cfs=cfs       
             
        for nr,wd in cfs:
            t=s.tred.column(nr,
                            width=wd,
                            anchor=s.anc,                            
                            )
        s.clearTred()
        s.liftWidget('tred')
        
        if not dats:
            
            showerror('Erro','Não há dados para leitura ler:\n "%s"'%s.path)
            return

        
        dats=[s.tredCols]+dats
        s.dcv=dcColVals(dats)
        s.tis=[]
        
        s.dkv=dict()        
        s.dtd=dict()

        if dats[0]==s.tredCols:
            dats=dats[1:]
      
        for ic,line in enumerate(dats):

            vals=[]
            for le in line:
                
                try:
                    le=unidecode(le)
                    if 'gmt-' in le.lower():
                        le=gmt2dma(le)
                    vals.append(le)
                    
                except:
                    vals.append(le)

            ti=s.tred.insert('',END,values=vals,tag='bl')
            s.dkv[line[0]]=vals
            s.key=line[0]
            
            s.dtd[ti]=vals
            s.tis.append(ti)
            
            if ic%1000==0:
                s.stb.config(text=ic+1,foreground='red')                
                s.tred.tag_configure('bl',foreground='blue')
                s.tred.update()      

        vls=list(s.dtd.values())
        
        eos=[e[0] for e in vls]
        s.figs=filtrAny(eos)

        s.stb.config(text=len(s.dtd),foreground='blue')
        s.stc.config(text=s.lift)        

        src=s.path   
        ext=splitext(src)[1]
        lnd=len(dats)
        
        try:
            fsz=formatedSize(getsize(src))
        except:
            fsz=line[1]
        cls=len(line)
        
        try:
            dta=invDat(getMtime(src))
        except:
            try:
                dta=line[2]
            except:
                pass
        
        s.adjustColumnsWidth()
        
            
    def saveFromTred(s):

        tis=s.tred.get_children()
        dst=s.dst[:]
        vls=[s.tred.item(ti,'values') for ti in tis]        
        s.writeVals(dst,vls)

    def writeVals(s,dst,vls):        

        dsn,ext=splitext(dst)
        s.dst=dst
        
        if s.lift=='tree':
            tab=[s.columns]+vls
            
        elif s.lift=='tred':
            tab=[s.tredCols]+vls
            
        dcv=dcColVals(tab)
        pdf=pd.DataFrame(dcv)
        ext=splitext(s.dst)[1]
       
        if ext=='.html':
            s.salvarHtml()

        elif ext=='.dkv':
            s.salvarDkv()
            
        else:
            
            if ext=='.json':
                pj=pdf.to_json(orient='records')
                jl=json.loads(pj)
                with open(s.dst,'w') as op:
                    json.dump(jl,op,indent=4)
                            
            elif ext=='.csv':
                pdf.to_csv(s.dst,index=False)            
                
            elif(ext=='.xml'):                
                s.tredCols=(e.replace(' ','_') for e in s.tredCols)
                pdf.to_xml(s.dst)

            elif ext=='.xlsx':
                pdf.to_excel(s.dst,index=False)

            elif ext in ['.txt','.txd','.dtx']:
                s.toDataText()

            elif ext in ['.pkl','.pkd']:
                pdf.to_pickle(s.dst)            

            s.path=s.dst[:]
            s.datas[s.path]=vls
            s.aqRead2Tred(s.path)

    def adjustColumnsWidth(s):        

        '''font=tkFont.Font(font=s.tex['font'])  # get font associated with Text widget
        tab_width = font.measure(' ' * 4)  # compute desired width of tabs
        s.tex.config(tabs=(tab_width,))  # configure Text widget tab stops

        font=('Segoe UI',13),
        tab_width = font.measure(' ' * 4)'''

        s.dcs=[]
        wid=s.ms.winfo_width()

        sm=[]
        cm=[]
        for cl in  s.dcv:
            vs=[e for e in s.dcv[cl]]            
            ls=[len(str(e)) for e in vs]

            md=sum(ls)/len(ls)           
            sm.append(md)
            cm.append((cl,md))
            
        tm=sum(sm)
        cs=s.tredCols
        cfs=[]

        for cl,md in cm:            
           
            wd=round(md*wid/tm)
            
            if wd>250:
                wd=250
            if wd<10:
                wd=10

            cfs.append(['#%d'%int(listind(cs,cl)+1),wd])

        dt=s.pdf.dtypes
        ds=[str(d) for d in dt]

        ns,ws=list(zip(*cfs))

        for nr,wd in cfs:
            
            ic=ns.index(nr)
            
            if 'object' in ds[ic]:
                anc=W
            else:
                anc=E
            if  containAny(cs[ic].lower(),['time','data']):
                anc=CENTER
                wd=40
                
            s.tred.column(nr,
                          width=wd,
                          anchor=anc,                            
                          )
        cs=s.tredCols
        for col in cs:            
            
            ic=cs.index(col)
           
            if 'object' in ds[ic]:                
                anc=W
            else:
                anc=E
                
            if containAny(cs[ic].lower(),['time','data']):
                anc=CENTER
                                    
            s.tred.heading(col,
                           text=col,
                           anchor=anc,
                           command=Fc(s.sortTred,col),
                           )

    def saveFromTred(s):

        tis=s.tred.get_children()
        
        vls=[s.tred.item(ti,'values') for ti in tis]
        dst=s.path[:]
        s.writeVals(dst,vls)

    def writeVals(s,dst,vls):

        dsn,ext=splitext(dst)
        s.dst=dst
        tab=[s.columns]+vls
        dcv=dcColVals(tab)
        pdf=pd.DataFrame(dcv)
        ext=splitext(s.dst)[1]
       
        if ext=='.html':
            s.salvarHtml()

        elif ext=='.dkv':
            s.salvarDkv()
            
        else:
            
            if ext=='.json':
                pj=pdf.to_json(orient='records')
                jl=json.loads(pj)
                with open(s.dst,'w') as op:
                    json.dump(jl,op,indent=4)
                            
            elif ext=='.csv':
                pdf.to_csv(s.dst,index=False)            
                
            elif(ext=='.xml'):                
##                s.tredCols=(e.replace(' ','_') for e in s.tredCols)
                pdf.to_xml(s.dst)#,attr_cols=list(s.tredCols))

            elif ext=='.xlsx':
##                s.tredCols=(e.replace(' ','_') for e in s.tredCols)
                pdf.to_excel(s.dst)#,attr_cols=list(s.tredCols))

            elif ext in ['.pkl','.pkd']:
                pdf.to_pickle(s.dst)            

            s.src=s.dst[:]
            s.aqRead2Tred(s.src)

    def acrescentarDiretorioAosFavoritos(s):

        if s.path!='':
            s.favoritHist[s.path]=agora()

    def qtdFigs(s):        

        ak=askstring('Quantidade',
                     'Entre com a quantidade de figuras,\n Total: %d figs'%len(s.figs),
                     initialvalue='%d %d'%(s.ini,s.end))
        
        if ak:
            sp=ak.split()
            s.ini,s.end=int(sp[0]),int(sp[1])
            
            fgs=s.figs[s.ini:s.end]
            s.figs2frame(fgs)

    def setOrdFigs(s,od):        

        a,c,d,t=['Original',
                 'Alfa crecente',
                 'Alfa decrescente',
                 'aleatória',
                 ]
        
        fgs=s.figs[:]
        
        if od==a:
            fgs=s.orig
            
        elif od==c:
            fgs.sort()
            
        elif od==d:
            fgs=figs[::-1]
            
        elif od==t:
            shuffle(fgs)

        s.figs=fgs


    def prettify(s):
        
        if splitext(s.path)[1] in ['.htm','.html']:
                        
            ct=s.tex.get('1.0',END)
            bs=BeautifulSoup(ct,features='lxml')

            pt=bs.prettify().strip()
            
            s.wTx.settext(pt)
            s.tex.update()
            ic=s.tex.index(END)
            ln,cl=ic.split('.')
            s.stb.config(text=ln)

    def keyPress(s,ev):

        Ks=ev.keysym
        
        if ev.widget==s.tree:
            
            if Ks=='space':
                Fc(s.nearFigs,1)
                
            elif Ks=='KeySpace':
                Fc(s.nearFigs,-1)

            else:            
            
                s.tree.focus()
                tis=s.tree.get_children()
                fs=[s.dtx[ti] for ti in tis]

                dic=dict()
                for fn in fs:
                    bn=basename(fn)
                    x=bn[0].lower()
                    ti=s.dxt[fn]
                    
                    if x in dic:
                        dic[x]+=[ti]
                    else:
                        dic[x]=[ti]
                        
                lt=ev.keysym.lower()
                if lt in dic:
                    ics=dic[lt]
                    
                    try:
                        s.ic=near(ics,s.ic)
                    except:
                        s.ic=ics[0]                
                        
                    ix=ics.index(s.ic)
                    ti=ics[ix]
                    
                    s.tree.selection_clear()
                    s.tree.selection_set(ti)
                    s.tree.focus(ti)
                    s.tree.see(ti)

        elif s.lift in ['frame','canvas']:
            
            if Ks=='space':
                Fc(s.nearFigs,1)
                
            elif Ks=='KeySpace':
                Fc(s.nearFigs,-1)


    def setMet(s,met):
        
        s.met=met
        s.expandirUrl()

    def expandirUrl(s,url=None,met=None):

        if met==None:
            met=s.met
            
        if url==None:
            url=s.ms.clipboard_get()
            s.url=url.strip()
            
 
        s.ent.xview(END)

        if all(e in url for e in list('[-]')):
            lks=desdob(url)

        else:
            mf=[
                ('Span figs'   ,s.spanFigs),
                ('splitLinks'  ,splitLinks),
                ('reqLinks'    ,reqLinks),
                ('fig2Links'   ,fig2Links),            
                ('figLS'       ,figLS),
                ('getLinks'    ,getLinks),           
                ('splitSources',splitSources),
                ('soupSources' ,soupSources),                        
                ('dobFig2Links',dobFig2Links),
                ('concUrlFigs' ,concUrlFigs),
                ]

            for(mt,fc) in mf:
                if met==mt:                
                    func=fc       
          
            lks=func(url)
            
            
        
        figs=filtrAny(lks)        
        
        if len(figs)>=0:
            
            s.ent.xview(END)                
            s.figs=figs
            s.figs2frame(figs)
            
            s.sta.config(text='This album is NOT saved yet')
            s.stb.config(text='%d a %d de %d'%(s.ini,s.end,len(figs)))
            s.stc.config(text='')
            s.std.config(text='NOT SAVED')            

            s.salvarConfigs()
            s.wBx.setlist(figs)
            s.wTx.settext('\n'.join(figs))

        if isUrl(url):
            s.expandHist[url]=agora()
            s.salvarConfigs()

    def proExpand(s,met=None):

        s.path=''

        if met==None:
            s.met='getLinks'        

        gc=s.ms.clipboard_get()
        sp=gc.split('\n')
        sp=singli(sp)
      
        sp=[et for et in sp if et!='']
        
        if len(sp)<=2:
            url=sp[0]
            s.url=url
            
            s.expandirUrl(url,met=s.met)
            url=unquote(url)
            
            s.sve.set(url)
            s.entryIn()
            s.tit.config(text=url)


        else:
            try:
                tx='\n'.join(sp)
                s.wTx.settext(tx)
                
                s.figs=filtrAny(sp,['-file:///'])
                s.showFigs(0)

                url=splitext(s.figs[0])[0]
                s.tit.config(text=url)                

            except:
                makeHtml(sp)
        

    def rollUrl(s,vl):

        ur=''
        sv=s.sve.get()

        if sv.startswith('http'):

            ie=s.ent.index(INSERT)
            
            ie=ie%len(sv)        
                
            dgs='0123456789'
            
            act=sv[ie]
           
            if act in dgs:
                
                ic=dgs.index(act)
                ic+=vl
                ic=ic%10
                
                nw=dgs[ic]
                bf=sv[:ie]
                af=sv[ie+1:] 
                ur=bf+nw+af
                
                

                im=urlImg(ur)
                s.toCanvas(ur,im)

                s.entryIn()

                s.sve.set(ur)
                
                s.ent.icursor(ie)


    def info(s):

        pt=s.path
        fg=s.fig
        lf=s.lift
        gs=len(s.figs)
        ni=s.ini
        nd=s.end
        
        lk=len(s.fgKv)
        lz=len(s.fgZp)
        lp=len(s.fgPt)
        fs=len(s.fis)

        st='''
Path:
%s

Fig:
%s

Lift: %s
Figs: %d

Ini: %d
End: %d

fgKv: %d
fgZp: %d
fgPt: %d

fis: %d


'''%(pt,fg,lf,gs,ni,nd,lk,lz,lp,fs)

        s.tpi.deiconify()
        s.wTi.settext(st)
        

    def criarInfoText(s):

        s.tpi=Toplevel()
        s.tpi.withdraw()
        s.tpi.title('Informações gerais')
        s.tpi.protocol('WM_DELETE_WINDOW',Fc(s.tpi.withdraw))

        s.wTi=Pmw.ScrolledText(s.tpi,
                               text_font=font1,
                               text_foreground='blue',
                               text_wrap='char',
                               )
        grid1(s.wTi)

        bt=Button(s.tpi,
                  text='Ok',
                  command=Fc(s.tpi.withdraw))
        bt.grid(row=2,column=1)        

        s.tpi.geometry('632x367+246+117')



    def verArquivosTemporarios(s):

        cd=os.getcwd()
        os.startfile(pjoin(cd,'temp'))

    def limparTemps(s):

        clearTrash()
        clearTemp()
        

    def recomporArquivo(s):        

        if s.lift=='canvas':

            s.scrFig=s.fig

            pt=s.fgPt[s.fig]            
            ex=fileExt(pt)

            if ex=='':
                ff=folderFiles(pt)
                zs=filtrEnd(ff,['.zip'])

                for zp in zs:
                    zf=ZipFile(zp)
                    ns=zf.namelist()
                    if s.fig in ns:
                        s.mostrarZip(zp)
                        break                           
                       
            elif ex in dkvExt:
                
                kv=openDkv(pt)
                di=list(kv.items())                
                
                for ky,vl in di:
                    for fg in vl:
                        if fg==s.fig:                            
                            fgs=kv[ky]
                            s.figs2frame(fgs)
                            return

            elif s.fig in s.fgKv:
                kv=s.fgKv[s.fig]
                s.mostrarDkv(kv)
                            
            elif s.fig in s.fgZp:
                zp=s.fgZp[s.fig]
                s.mostrarZip(zp)

            elif isUi(s.fig):
                fgs=spanFigs(s.fig)[1]
                s.figs2frame(fgs)

            else:

                ur=nm2ur(s.fig)
                if isUi(ur):
                    fgs=spanFigs(ur)[1]
                    s.figs2frame(fgs)
                    
                
    def showMask(s):

        if s.lift=='tred':

            tis=s.tred.get_children()
            
            try:
                s.ti=s.tred.selection()[0]
            except:
                s.ti=tis[0]

            vals=s.tred.item(s.ti,'values')
            zp=zip(s.tredCols,vals)  

        elif s.lift=='tree':

            tis=s.tree.get_children()
            
            try:
                s.ti=s.tree.selection()[0]
            except:
                s.ti=tis[0]
                
            'vals=[bn,ex,fs,fz,ft,bs]'
            'item=[bn,ex,fs,sz,mt,bs,dn,tg]'
            
            bn,ex,fs,sz,mt,bs,dn,tg=s.dti[s.ti]
            mt=invDat(mt)
            fz=fmSize(sz)
            bs=bs.strip()
            sp=bs.split(' ')
            sp=[et for et in sp if len(et)>0]
            bs='  '.join(sp)
            vals=[pjoin(dn,bn),ex,fs,fz,mt,bs]
            
            zp=zip(s.columns,vals)        

        try:
            s.topProp.deiconify()
        except:
            s.topProp=s.topProp=Toplevel()
            
        s.topProp.title('Propriedades')
        s.topProp.protocol('WM_DELETE_WINDOW',Fc(s.topProp.destroy))

        s.topProp.bind('<Right>',Fc(s.nearLines,1))
        s.topProp.bind('<Left>',Fc(s.nearLines,-1))

        s.topProp.focus()

        tit=Label(s.topProp,
                  height=3,
                  font=('Arial',14),
                  text='Propriedades',
                  )
        tit.grid(row=0,column=1)
        
        s.wSf=Pmw.ScrolledFrame(s.topProp,
                                borderframe=1,
                                borderframe_relief='flat',
                                frame_background='yellow',
                                )     
        grid1(s.wSf)

        s.frm=s.wSf.component('frame')
        

        ls=[len(str(e)) for e in s.columns]
        wl=max(ls) 
   
        s.propVars=[]  
        
        for ic,(cl,dt) in enumerate(zp):
            
            tve=StringVar()            
            tvl=StringVar()
            
            tve.set(dt)
            tvl.set('%s:'%str(cl).capitalize())            
            
            ef=Pmw.EntryField(s.frm,                              
                              labelpos='w',
                              label_width=wl,
                              label_textvariable=tvl,
                              label_anchor='e',                              
                              entry_textvariable=tve,
                              entry_width=100,
                              entry_font=('Lucida console',13),
                              entry_foreground='blue',
                              )
            
            s.propVars.append(tve)            
            
            ef.grid(row=ic+1,column=1)
            en=ef.component('entry')
            lb=ef.component('label')
            lb.grid_configure(ipadx=10)
            
            en.bind('<Double-1>',s.toLauncher)
            en.bind('<Insert>',Fc(s.alterarLinha))
            lb.bind('<3>',s.popLabel)

        frBt=Frame(s.topProp)
        frBt.grid(row=ic+2,column=1,pady=15)

        bts=['<<',
             '>>',
             'Alterar',             
             'Browse',
             'Texto',
             'Voltar',
             ]
        
        fcs=[Fc(s.nearLines,-1),
             Fc(s.nearLines,1),
             s.alterarLinha,             
             s.startFile,
             s.propriedText,
             Fc(s.topProp.destroy),
             ]

        bls=['Mostrar linha anterior',
             'Mostrar linha seguinte',
             'Alterar linha',             
             'Mostrar no browser',
             'Mostrar texto',
             'Voltar tela principal',
             ]
        
        zpb=list(zip(bts,fcs,bls))

        for ic,(lb,fc,bo) in enumerate(zpb):
            bt=Button(frBt,
                      text=lb,
                      command=fc,
                      width=6,
                      )
            bt.grid(row=0,column=ic,padx=2)
            bl=Pmw.Balloon(xoffset=-10,yoffset=10)
            bl.bind(bt,bo)
            bt.bind('<Return>',Fc(fc))
            
        s.ms.bind('<Right>',Fc(s.nearFiles,1))
        s.ms.bind('<Left>',Fc(s.nearFiles,-1))

        s.topProp.bind('<MouseWheel>',s.tredWheel)
        s.topProp.geometry('800x300+30+20')
        
        centralizar(s.topProp)

    def toLauncher(s,ev):
        
        gw=ev.widget.get()
        try:
            os.startfile(gw)
        except Exception as ex:
            print('toLaucher')
            print(gw)
            print(ex)
            print()

    def getStrVars(s):

        rs=[]
        for sv in s.propVars:
            gt=sv.get()            
            rs.append(gt)
        return rs

    def near(s,vl):
        
        s.ti=near(s.tis,s.ti,vl)
        vs=s.dtd[s.ti]
        
        zp=list(zip(s.propVars,vs))

        for sv,vl in zp:
            sv.set(vl)
            
        s.tred.selection_clear()
        s.tred.selection_set(s.ti)
        s.tred.see(s.ti)


    def alterarLinha(s):

        ti=s.ti
        vs=s.getStrVars()
        s.tred.item(ti,values=vs)
        s.dtd[ti]=vs
        
    def popLabel(s,ev):

        wg=ev.widget

        mu=Menu(tearoff=0)
        li=['Remover coluna',]
        
        fc=[Fc(s.removerColuna,ev),            
            ]
        
        zp=list(zip(li,fc))
        for lb,cm in zp:
            mu.add_command(label=lb,command=cm)

        mc=Menu(mu,tearoff=0)
        mu.add_cascade(label='Anchor',menu=mc)

        lc=[('Esquerda',W),
            ('Centro',CENTER),
            ('Direita',E),]

        for lb,anc in lc:
            mc.add_command(label=lb,command=Fc(s.setColAnchor,ev,anc))        

        mu.post(ev.x_root,ev.y_root)

    def nearLines(s,vl):

        if s.lift=='tred':
            s.nearTred(vl)
            
        elif s.lift=='tree':
            s.nearData(vl)

    def nearTred(s,vl):

        if s.lift=='tred':
            s.tis=s.tred.get_children()
            
            s.ti=near(s.tis,s.ti,vl)
            vs=s.dtd[s.ti]
            
            zp=list(zip(s.propVars,vs))

            for sv,vl in zp:
                sv.set(vl)
                
            s.tred.selection_clear()
            s.tred.selection_set(s.ti)
            s.tred.see(s.ti)

    def nearData(s,vl):

        if s.lift=='tree':

            s.tis=s.tree.get_children()
        
            s.ti=near(s.tis,s.ti,vl)
    
            bn,ex,fs,sz,mt,bs,dn,tg=s.dti[s.ti]
            mt=invDat(mt)
            fz=fmSize(sz)
            bs=bs.strip()
            sp=bs.split(' ')
            sp=[et for et in sp if len(et)>0]
            bs='  '.join(sp)
            vs=[pjoin(dn,bn),ex,fs,fz,mt,bs]
            
            zp=list(zip(s.propVars,vs))

            for sv,vl in zp:
                sv.set(vl)
                
            s.tree.selection_clear()
            s.tree.selection_set(s.ti)
            s.tree.see(s.ti)
        

    def startFile(s):

        if s.lift=='tree':
            s.fromTree()
            
        elif s.lift=='tred':
            s.fromTred()

        s.topProp.destroy()

    def propriedText(s):

        zp=zip(s.columns,s.propVars)
        ls=[len(et) for et in s.columns]
        mx=max(ls)+2

        li=['\n']
        for cl,tv in zp:            
            gt=tv.get()
            st='%xs : %s'.replace('x',str(mx))%(cl,gt)
            li.append(st)

        tx='\n'.join(li)
        tx=unidecode(tx)
        s.wTx.settext(tx)
        s.liftWidget('text')
        s.tex.config(fg='blue')

    def tredWheel(s,ev):

        if s.wSf.winfo_ismapped():

            dt=ev.delta

            if   dt>0:vl=-2
            elif dt<0:vl= 2

            s.wSf.yview('scroll',vl,'units')

    def spanFigs(s,url=None):

        sv=s.ms.clipboard_get()
        st,ci=spanFigs(sv)
        s.sve.set(st)
        s.ent.xview(END)
        return ci

    def spanCanvas(s):

        global st,ci

        if isfile(s.fig):
            fig=nm2ur(s.fig)

        elif 'http' in s.fig:
            fig=s.fig[:]

        elif s.fgPt[s.fig].endswith(tuple(zipExt)):
            fig=nm2ur(s.fig)       
            

        st,ci=spanFigs(fig)
        s.sve.set(st)
        s.ent.xview(END)
        s.ini,s.end=0,s.qtd
        
        s.figs2frame(ci)
        if all(e in st for e in list('[-]')):
            ur=desdob(st)[0]
        if isUrl(st):
            s.expandHist[st]=agora()
        

    def copiarPath(s):

        

        tp=Toplevel()
        lb=Label(tp,
                 text='Copiado:\n%s\n'%s.path,
                 fg='red',
                 font=('Times',12),
                 )
        lb.grid(padx=20, pady=20)
        tp.update()
        centralizar(tp)
        tp.after(3000,Fc(tp.destroy))       


    def verFigsHist(s):        

        fis=[]        

        ftm=list(s.fgTmIm.items())
        ax=[]
        for fg,(tm,im) in ftm:
            ax.append((fg,tm,im))
        ax=icSort(ax,1)
        ax=ax[::-1]
        s.fis=[(fg,im) for (fg,tm,im) in ax]                
        
        s.preparFrame()
        
        for (fg,im) in s.fis:
            s.addThumb(fg,im)
            
        s.figs,s.imgs=list(zip(*s.fis))
        s.tit.config(text='Histórico de figuras')
        s.stb.config(text='%d figs'%(len(s.fis)))

        try:
            cv=s.dfc[s.scrFig]
            cy=cv.winfo_y()
            fy=s.frame.winfo_height()
            alt=cy/fy
            s.wFr.yview('moveto',alt)            
            cv.configure(highlightbackground='blue')

        except Exception as ec:
            pass
        
    def getFgIm(s,fg):        
            
        if fg.startswith('http'):
            
            try:
                ct=urlContent(fg)
                ib=io.BytesIO(ct)
                im=Image.open(ib)                                       
                wd,he=im.size
                if wd>350 and he>350:
                    s.fis.append((fg,im))

            except Exception as ex:
                s.nfi.append(fg)

        elif s.fgPt[fg].endswith(tuple(zipExt)):

            try:
                zf=ZipFile(s.fgPt[fg])
                rd=zf.read(fg)
                ib=io.BytesIO(rd)        
                im=Image.open(ib)
                wd,he=im.size
                if wd>350 and he>350:
                    s.fis.append((fg,im))
            except:
                s.nfi.append(fg)

        elif isfile(fg):

            try:                
                im=Image.open(fg)
                s.fis.append((fg,im))
            except:
                s.nfi.append(fg)

    def executar(s):

        gv=s.sve.get()        
        exec(gv)
        s.wEf.lift()


    def isOver(s):
        
        sw=s.ms.winfo_screenwidth()
        sh=s.ms.winfo_screenheight()       
        
        s.ms.update_idletasks()
        
        ww=s.ms.winfo_width()
        wh=s.ms.winfo_height()
        
        
        if (sw,sh)==(ww,wh):
            return True
        else:
            return False


    def togleScreen(s):      

        if s.isOver():
            s.telaPadrao()
        else:
            s.telaCheia()

    def telaPadrao(s):

        s.ms.state('normal')
        s.ms.geometry('930x450+100+40')
        s.ms.overrideredirect(0)
        s.frs.grid()        
        s.fri.grid()
        s.ms.config(menu=s.Men)
        
    def telaCheia(s):
        
        s.ms.state('zoomed')
        s.ms.overrideredirect(1)
        s.frs.grid_remove()        
        s.fri.grid_remove()
        s.ms.config(menu='')


    def zipparDkv(s):

        kvEnd=''

        if s.lift=='tree':
            ti=s.tree.focus()
            kv=s.tree.item(ti,'values')[0]
            fp=pjoin(s.path,kv)
            kvEnd=fp

        else:            
        
            kvEnd=s.tit['text']

        if isfile(kvEnd):
                    
            dst=askdirectory(initialdir='')
        
            if dst:                 
                
                op=open('dkvEnd.pkl','wb')
                dump((kvEnd,dst),op)
                op.close()
                
                sleep(0.5)
                os.startfile('dkvZipper.py')

    def salvarTodasAsFiguras(s):

        ds=askdirectory(initialdir='')
        if ds:
            op=open('downFigs.pkl','wb')
            dump((s.figs,ds),op)
            op.close()
            os.startfile('downFigs.pyw')

    def dkvs5mais(s):

        kv=s.dkv
        di=list(s.dkv.items())
        d2=[(k,v) for (k,v) in di if len(v)>=5]
        s.dkv=dict(d2)
        s.dkv2tred()

    def removerRepetidosNoDkv(s):

        s.dkv=singleDkv(s.dkv)
        di=list(s.dkv.items())
        ax=dict()
        for ky,vs in di:
            if len(vs)>=5:
                ax[ky]=singli(vs)
        s.dkv=ax
        if fileExt(s.path) in dkvExt:
            salvarDkv(s.dkv,s.path)
        s.mostrarDkv(s.path)

    def url2links(s):

        if s.lift=='tred':
            ti=s.tred.focus()
            vs=s.tred.item(ti,'values')
            ic=listind(vs,'http')
            ur=vs[ic]
            s.url=ur
            s.expandirUrl(ur)

        elif s.lift=='text':
            ur=s.getTextLine()
            s.expandirUrl(ur)

    def iniSlider(s):
        
        s.ms.after(2000,s.loop)        

    def loop(s):

        s.nearFigs(1)
        s.tid=s.ms.after(2000,s.loop)        

    def endSlider(s):

        if s.tid!=None:
            s.ms.after_cancel(s.tid)
        s.tid=None

        
            
if __name__=='__main__':          

    tk=Tk()
    s=Mtreeview(tk)
##    tk.mainloop()
        
