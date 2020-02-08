~~~mysql
CREATE OR REPLACE PROCEDURE "IMUSICSMS"."INSERTSMSRECEPTION" (
  returnval OUT VARCHAR2, --返回值，如果正常则是当前记录id,若果错误则是错误描述
  vmessage  IN  VARCHAR2,--短信内容
  vsmsServiceActivationNumber  IN  VARCHAR2,
  vsenderaddress  IN  VARCHAR2,
  vregistrationIdentifier  IN  VARCHAR2,
  vtransactionid IN  VARCHAR2,--业务流水号
  vlinked  IN  VARCHAR2,--事务关联ID
  vspid  IN  VARCHAR2,
  vsan  IN  VARCHAR2
) IS
vprodid varchar2(21);--sms产品id
vcode   varchar2(21);--sms产品业务id
vcharging varchar2(5);--sms产品的计费数,单位 'CNY'
vcommondtype number;--sms产品的类型 1订购 2点播
vmincharging number;--在订购命令相似时,最小的sms产品计费数
vcount number;--存储订购指令能匹配到的sms产品的记录数量
BEGIN
  
  --获取订购指令能匹配到的sms产品的记录数量
  select count(*) into vcount from  v_isag_product tt where  tt.commondorder like vmessage||'%';
  if vcount=0 then
     --没有订购指令匹配时,使用任意匹配0元点播sms产品信息记录
     select t.proid,t.code,t.charging,t.commondtype into vprodid, vcode, vcharging, vcommondtype
     from v_isag_product t where t.commondorder='*';
  else 
     --在多条可能的匹配记录中挑选 计费最少 sms产品类型数最小 的记录,并存储此时最少计费和最小产品类型数
      select to_number(tt.charging),min(to_number(tt.commondtype))into vmincharging,vcommondtype
      from v_isag_product tt
      where to_number(tt.charging)=(select min(to_number(t.charging)) from v_isag_product t 
                                   where t.commondorder like vmessage||'%')
            and tt.commondorder like vmessage||'%'
      group by tt.charging;
      --利用上条sql匹配的结果,选出 sms产品id,sms产品业务id,sms产品的计费数
      select t.proid,t.code,t.charging into vprodid, vcode, vcharging
      from v_isag_product t 
      where to_number(t.charging)=vmincharging and to_number(t.commondtype)=vcommondtype
            and  t.commondorder like vmessage||'%';
  end if;
  
  --获取插入上行表的此条记录的预备使用的id     
  select to_char( ISAG_SMS_DELIVER_INFO_SEQ.NEXTVAL) into returnval from dual;
  --插入上行数据到上行表中
  insert into isag_sms_deliver(CZID,MESSAGE,SMSSERVICEACTIVATIONNUMBER,SENDERADDRESS,REGISTRATIONIDENTIFIER,
                          TRANSACTIONID,LINKED,SPID,SAN) values(returnval,vmessage,vsmsServiceActivationNumber,
                          vsenderaddress,vregistrationIdentifier,vtransactionid,vlinked,vspid,vsan);
 --插入上行指令匹配的产品信息到下行表,通知订购客户,完成计费
  insert into isag_sms_submit(czid,message,senderaddress,reciveaddress,transactionid,linked,san,productid,
              code,oa,receiptrequest,charging) 
		values(isag_sms_submit_info_seq.nextval,'指令:'||vmessage||' 产品id:'||vprodid,vsmsServiceActivationNumber,
    vsenderaddress,vtransactionid,vlinked,vsan,vprodid,vcode,vsenderaddress,'1',vcharging);
    -- correlator=to_char(sysdate,'MMddhh24miss')||isag_sms_submit_info_seq.currval,
  exception
   when others then
    returnval :=sqlerrm;
    rollback;
END;
~~~

