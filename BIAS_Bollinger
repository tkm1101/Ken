import pandas as pd
import oandapy
import os
import numpy as np

account_number=os.getenv("account_number")
access_token=os.getenv("access_token")

def invested():
    trade_status=trade_status=oanda.get_trades(account_id=account_number)['trades']
    
    if trade_status == []:
        return 'empty'
    elif trade_status != []:
        if trade_status[0]['side'] == 'buy':
            return 'buy'
        elif trade_status[0]['side'] == 'sell':
            return 'sell'

def ABIAS(data):
    abias=[]
    for i in data:
        if i>0.04:
            abias.append(i)
        elif i<0.04:
            abias.append(0.04)
    return abias

def BBIAS(data):
    bbias=[]
    for i in data:
        if i< -0.04:
            bbias.append(i)
        elif i > -0.04:
            bbias.append(-0.04)
    return bbias


while True:

    
    
    oanda = oandapy.API(environment="practice" , access_token=access_token)
    response = oanda.get_history(instrument="UK100_GBP",
                                      granularity='M1',
                                      count = 500)
    
    df2=pd.DataFrame(data=response['candles'],index=None)
    
    

    df_midclose=pd.DataFrame(data=(df2['closeAsk']+df2['closeBid'])/2,columns=['MidClose'])
    df_midlow=pd.DataFrame(data=(df2['lowAsk']+df2['lowBid'])/2,columns=['MidLow'])
    df_midhigh=pd.DataFrame(data=(df2['highAsk']+df2['highBid'])/2,columns=['MidHigh'])
    
    gh=df_midhigh['MidHigh'].rolling(window=9).max()
    gl=df_midlow['MidLow'].rolling(window=9).min()
    a=(gh+gl)/2
    gzz=a.ewm(span=3).mean()
    bias=(df_midclose['MidClose']/gzz-1)*100
    bias_formal=np.nan_to_num(bias)


    ABIAS_formal=pd.DataFrame(data=ABIAS(bias_formal),columns=['ABIAS'])
    params1=ABIAS_formal['ABIAS'].rolling(window=20).max()
    shh=params1.ewm(span=3).mean()

    BBIAS_formal=pd.DataFrame(data=BBIAS(bias_formal),columns=['BBIAS'])
    params2=BBIAS_formal['BBIAS'].rolling(window=20).min()
    xll=params2.ewm(span=3).mean()
    
    
    #Start placing trade
    if invested()== 'empty':
        if bias_formal[499] < xll[499]:
            oanda.create_order(account_id=account_number,
                              instrument='UK100_GBP',
                              units=1,
                              side='buy',
                              type='market')
        elif bias_formal[499]> shh[499]:
            oanda.create_order(account_id=account_number,
                              instrument='UK100_GBP',
                              units=1,
                              side='sell',
                              type='market')
            
    
    elif bias_formal[499] > shh[499] and invested() == 'buy':
        oanda.close_position(account_id=account_number,instrument='UK100_GBP')
            
    
    elif bias_formal[499] < xll[499] and invested() == 'sell':
        oanda.close_position(account_id=account_number,instrument='UK100_GBP')
