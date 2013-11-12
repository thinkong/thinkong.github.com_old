---
layout: post
category : .net
tagline: "json to sql"
tags : [json, .net, c#, sql]
---
{% include JB/setup %}

얼마나 필요할지는 모르지만... 일단 만들어 봤다...

Tabled.cs
```C#
namespace jsonutil
{
    class TableD
    {
        public static Dictionary<string, TableD> oDic = new Dictionary<string, TableD>();

        public static void printExpandoObject(ExpandoObject o, int ilvl = 0, string parent = "", string myname = "")
        {
            var list = o.ToList();
            TableD oTab = new TableD();
            oTab.foreignkey = parent;
            oTab.tablename = myname;
            foreach (var e in list)
            {
                if (e.Value.GetType().IsArray)
                {
                    ExpandoObject[] eArray = (ExpandoObject[])e.Value;
                    foreach (var a in eArray)
                    {
                        printExpandoObject(a, ilvl + 1, myname, e.Key);
                    }
                }
                else
                {
                    var eVal = e.Value;
                    if (eVal.GetType() == typeof(ExpandoObject))
                    printExpandoObject((ExpandoObject)eVal, ilvl + 1, myname, e.Key);
                    else 
                    oTab.fieldlist.Add(e.Key, e.Value);
                }
            }
            string tname = parent + ":" + myname;
            if (!oDic.ContainsKey(tname))
            {
                oDic.Add(tname, oTab);
            }
            else
            {

                oDic[tname].Merge(oTab);
            }
        }
        public string tablename { get; set; }
        public string foreignkey { get; set; }
        public Dictionary<string, object> fieldlist = new Dictionary<string, object>();
        private Dictionary<Type, String> dataMapper
        {
            get
            {
                // Add the rest of your CLR Types to SQL Types mapping here
                Dictionary<Type, String> dataMapper = new Dictionary<Type, string>();
                dataMapper.Add(typeof(int), "BIGINT");
                dataMapper.Add(typeof(string), "NVARCHAR(500)");
                dataMapper.Add(typeof(bool), "BIT");
                dataMapper.Add(typeof(DateTime), "DATETIME");
                dataMapper.Add(typeof(float), "FLOAT");
                dataMapper.Add(typeof(decimal), "DECIMAL(18,0)");
                dataMapper.Add(typeof(Guid), "UNIQUEIDENTIFIER");
                dataMapper.Add(typeof(Int64), "BIGINT");

                return dataMapper;
            }
        }
        public void PrintTables()
        {
            System.Console.WriteLine("tablename:{0}", tablename);
            foreach (KeyValuePair<string, object> o in fieldlist)
            {
                System.Console.WriteLine("field:{0}, {1}", o.Key, o.Value.GetType().ToString());
            }
        }

        public void Merge(TableD oTab)
        {
            foreach (KeyValuePair<string, object> o in oTab.fieldlist)
            {
                if (!fieldlist.ContainsKey(o.Key))
                {
                    fieldlist.Add(o.Key, o.Value);
                }
            }
        }

        public string CreateTableScript()
        {
            System.Text.StringBuilder script = new StringBuilder();

            script.AppendLine("CREATE TABLE " + this.tablename);
            script.AppendLine("(");
                script.AppendLine("\t ID BIGINT NOT NULL PRIMARY KEY,");
                int i = 0;
                foreach (var field in fieldlist)
            //for (int i = 0; i < this.fieldlist.Count; i++)
                {
                //KeyValuePair<String, Type> field = this.fieldlist[i];
                    var oVal = field.Value;
                    var oType = "[unknown]";
                    try
                    {
                        oType = dataMapper[oVal.GetType()];
                    }
                    catch
                    {

                    }
                    script.Append("\t " + field.Key + " " + oType + ",");

                //if (i != this.fieldlist.Count - 1)
                //{
                //    script.Append(",");
                //}

                    script.Append(Environment.NewLine);
                    ++i;
                }

                script.Append("\t" + foreignkey + "_id BIGINT FOREIGN KEY REFERENCES " + foreignkey + "(" + "ID" + ")");

                script.Append(Environment.NewLine);
                script.AppendLine(")");
                return script.ToString();
            }
        }
    }
}
```

사용방법
```C#
// using jsonfx for example

string jsont = File.ReadAllText(@"somejsonfile.json");
var jsonreader = new JsonFx.Json.JsonReader();
var obj = (System.Dynamic.ExpandoObject)jsonreader.Read(jsont);
printExpandoObject(obj);
string allscript = "";
foreach (var o in oDic.Reverse())
{
    allscript += o.Value.CreateTableScript();
}

// allscript will contain multiple create queries.. 
// some errors will be there..(empty stuff. etc... )
// will be good for a starting point on json stuff
```