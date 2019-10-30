```
#! /usr/bin/python3

import xlrd
import xlsxwriter

def read_excel():
    workbook=xlrd.open_workbook(r'/sdata/scripts/python3-es/files/elk-user-info.xls')
    sheet_name=workbook.sheet_names()[0]
    sheet=workbook.sheet_by_index(0)
    #print (sheet.name,sheet.nrows,sheet.ncols)
    cols=sheet.col_values(0)
    return cols

def write_excel(value): 
    workbook = xlsxwriter.Workbook('/sdata/scripts/python3-es/files/elk-user-email.xls')
    worksheet = workbook.add_worksheet(u'sheet1')
    for n in range(len(value)):
        worksheet.write(n,0,value[n]+"@ingbaobei.com")
    workbook.close()
    print("写入完成")


if __name__ == '__main__':
    username=read_excel()
    write_excel(username)
```
