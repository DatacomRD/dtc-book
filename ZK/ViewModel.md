> # 共用 ViewModel 的介紹 #
> 這是輔助文件，請還是要記得看各 class 的 JavaDoc


目前蓋了這幾種 VM，族譜如下：

	`BaseViewModel`
		`BaseEntityViewModel`
			`BaseMaintainViewModel`


### BaseViewModel ###
沒有什麼特殊功能，就 ZK 範圍的功能來說，就是提供以程式觸發 MVVM 的 method，
例如 `notifyChange()`、`postCommand()` 等等。


### BaseEntityViewModel ###
開始跟 ZUL 有<strike>糾纏不清的</strike>關係了，所以先貼一下基本的頁面樣板：

```XML
	<window apply="org.zkoss.bind.BindComposer" viewModel="@id('vm') @init('________')" 
		title="${c:l('________')}" 
    	width="100%" height="100%" self="@define(content)" border="normal">
		
		<!-- 搜尋按鈕區 -->
		<caption>
			<button label="${c:l('button.search')}" onClick="@command('search')" />
			<button label="${c:l('button.resetQuery')}" onClick="@command('resetSearch')" />
		</caption>

		<vlayout vflex="1">
			<listbox model="@load(vm.listModel)" onSelect="@command('selectData')"
				mold="paging" autopaging="true" emptyMessage="${c:l('empty.message')}"
				vflex="1" sizedByContent="true" span="true">
				
				<auxhead form="@id('sfx') @load(vm.constraint) @save(vm.constraint, before='search')">
					<!-- 搜尋欄位，要跟 entity 欄位名稱一樣 -->
				</auxhead>
				
				<listhead>
					<!-- 欄位欄位欄位 -->
				</listhead>
				
				<template name="model">
					<listitem>
						<!-- 欄位欄位欄位 -->
					</listitem>
				</template>
			</listbox>

			<!-- 以下是編輯區，要對應 BaseMaintainViewModel -->
			
			<!-- 正常狀況下：37 是 25（groupbox 的 caption） + 12（grid 的 padding）。
				有 button 的 row 的高度會變成 40-->
			<groupbox height="${c:cat(37 + 32 * rowAmount, 'px')}" mold="3d"
				form="@id('fx') @init(vm.editorForm) @load(vm.currentData) @save(vm.currentData, before='save')">
				<caption label="${c:l('editor.title')}" visible="@load(vm.triggerStatus(fxStatus.dirty))" />			
				<grid visible="@load(vm.editorVisible)">
				
					<!-- 欄位欄位欄位 -->
				
				</grid>
			</groupbox>
				
			<hlayout>
				<div width="150px">
					<label value="@load(vm.status)" style="font-weight:bold;font-size:12pt;color:red" />
				</div>
				<hbox hflex="1" pack="center" align="center">
					<button onClick="@command('add')" disabled="@load(vm.disableAdd)" 
						label="${c:l('button.add')}" width="100px" />
					<button onClick="@command('save')" disabled="@load(vm.disableSave)"
						label="${c:l('button.save')}" width="100px" />
					<button onClick="@command('cancel')" disabled="@load(vm.disableCancel)" 
						label="${c:l('button.cancel')}" width="100px" />
					<button onClick="@command('delete')" disabled="@load(vm.disableDelete)"
						label="${c:l('button.delete')}" width="100px" />
				</hbox>
			</hlayout>
		</vlayout>
	</window>
```

當然，重點是跟 view model 有關的部份，
除了不能把 form binding 內的欄位抽到 form binding 的 id space 之外，
不然 component 跟 layout 都可以自由變化。

`BaseEntityViewModel` 假設這個 VM 是以某一個 entity 為主軸，也就是帶入到泛型的 class；
還有 view 會顯示這個 entity 的資料（們）。

`BaseEntityViewModel` （試圖）解決了一般狀態下的 search 功能，
基本上只要把 form binding（`sfx`）的 field 的名稱設定的跟 entity 的名稱相同，
`BaseEntityViewModel` 就會幫你組好 hql（包含搜尋指定區間的需求）。
另外也幫你作掉「重設搜尋條件」的功能。


#### tip ###
* `queryData()` 是沒有給任何搜尋條件的時候資料取得的方式。
	`queryData(String, Map)` 是有給搜尋條件的時候資料取得的方式。
* 重新載入資料：呼叫 `search()`。
* 搜尋條件要有預設值：override `afterResetSearch()`，把預設值的 key-value 塞進 `getConstraint()`。
	例如：

		@Override
		protected void afterResetSearch() {
			Date date = new Date();
			getConstraint().put("studyDate_from", date);
			getConstraint().put("studyDate_to", date);
		}

* 畫面載入時符合預設搜尋條件的資料：在 VM 的 constructor 呼叫 `afterResetSearch()`
	（當然要有 override `afterResetSearch()`）。


### BaseMaintainViewModel ###
`BaseMaintainViewModel` 繼承 `BaseEntityViewModel`，然後補上「編輯資料」的功能。
也就是說，`BaseMaintainViewModel` 會幫忙 hold 住跟編輯資料有關的 UI 狀態，
例如編輯區的狀態控制：

| 狀態     | 編輯區 | 新增 | 存檔 | 取消 | 刪除 |
|----------|--------|------|------|------|------|
| 初始狀態 | 關     | V    |      |      |      |
| 指定資料 | 開	    | V    |      |      | V    |
| 按下新增 | 開		| V    |      |      |      |
| 編輯中   | 開	    |      | V    | V	 |      |

「編輯中」 `fxStatus.dirty` 會是 true

另外像是編輯中但是又<strike>手賤</strike>去按「新增」時要使用者 confirm，
也是 `BaseMaintainViewModel` 的防守範圍。


### 子母 entity 的議題 ###
有兩個 entity：`Parent`、`Child`，一個 `Parent` 底下可以有多個 `Child`，
然後要在同一個頁面上同時維護 `Parent` 與 `Child` 的資料。

開兩個 VM：`ParentVM` 當中包含了 `ChildVM`。
`ChildVM` 的 `add` 等 command 不會觸發 `ParentVM` 的 command。

目前最大的 issue 是當 `ParentVM.selectData()` 觸發時，若是 `ChildVM` 的編輯區在修改中（dirty），
則沒有設計（也可能設計不出來）機制可以提醒使用者尚未儲存然後 rollback 回去的可能性。