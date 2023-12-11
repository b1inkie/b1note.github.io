## 添加配方

```lua
local Ingredient = GLOBAL.Ingredient
local TECH = GLOBAL.TECH

local function ModAtlas(ingredients,amount)
	local atlas = "images/inventoryimages/"..ingredients..".xml"
	return Ingredient(ingredients,amount,atlas)
end
local function ProductAtlas(product)
	local atlas = "images/inventoryimages/"..product..".xml"
	return atlas
end
local function isShownFn(...)
	for k,v in pairs({...}) do
		if v == "yes" then
			return true
		end
	end
	return false
end

--给MOD物品添加一个分类
local MOD_FILTER_TAG = "MYMOD_TAB"

AddRecipeFilter({
    name = "MYMOD_TAB",
    atlas = "images/mymodtab.xml",
    image = "mymodtab.tex"
})
STRINGS.UI.CRAFTING_FILTERS.MYMOD_TAB ="MYMOD TAB"

local recipe_all = {
	--[[
	{
		recipe_name = "gutknife_recipe_1", --配方ID
		ingredients = { --配方
			Ingredient("nightsword", 1),
			Ingredient("gears", 1),
		},
		tech = TECH.SCIENCE_ONE, --所需科技
		isOriginalItem = true, --是官方物品,不写则为自定义物品
		isShown = true, --不写或true显示配方, 其他值则不显示
		config ={ --其他的一些配置,可不写
			--制作出来的物品,不写则默认制作出来的预制物为食谱ID
			product = "gutknife", 
			--xml路径,不写则默认路径为,"images/inventoryimages/"..product..".xml" 或 "images/inventoryimages/"..recipe_name..".xml"
			atlas = "images/inventoryimages/gutknife.xml",
			--图片名称,不写则默认名称为 product..".tex" 或 recipe_name..".tex"
			image = "gutknife.tex",
			--制作出的物品数量,不写则为1
			numtogive = 40,
		},
		filters = {"MYMOD_TAB"} --将物品添加到这些分类中,可不写
	},
	]]
    ----------
    --官方物品
    ----------
    {
		recipe_name = "sewing_tape_2",
		isOriginalItem = true,
		ingredients = {
			Ingredient("livinglog", 2),
			Ingredient("silk", 4),
        },
		tech = TECH.NONE,
		config = {
			product = "sewing_tape",
			numtogive = 2,
		},
		filters = {"MYMOD_TAB"}
	},
    ---------
    --MOD物品
    ---------
    {
		recipe_name = "gutknife",
		ingredients = {
			Ingredient("nightsword", 1),
			Ingredient("gears", 1),
		},
		isShown = isShownFn(TUNING.CONFIG_GUTKNIFE_RECIPE),
		tech = TECH.SCIENCE_TWO,
	},
	----------------
	--STRUCTURE-----
	----------------
	{
		recipe_name = "mypot",
		ingredients = {
			Ingredient("log", 3),
			Ingredient("cutstone", 2),
		},
		tech = TECH.SCIENCE_ONE,
		config = {
			placer = "mypot_placer",
		}
	},
}

for k,_r in pairs(recipe_all) do
	if _r.isOriginalItem == nil then
		if _r.config == nil then
			_r.config = {}
		end
		if _r.config.atlas == nil then
			if _r.config.product ~= nil then
				_r.config.atlas = ProductAtlas(_r.config.product)
				_r.config.image = _r.config.product..".tex"
			else
				_r.config.atlas = ProductAtlas(_r.recipe_name)
				_r.config.image = _r.recipe_name..".tex"
			end
		end
	end
	if _r.filters == nil then
        if MOD_FILTER_TAG ~= nil then
		    _r.filters = {MOD_FILTER_TAG}
        else
            _r.filters = {}
        end
	end
	if _r.config == nil then
		_r.config = {}
	end
	if _r.isShown == nil or _r.isShown == true then
		AddRecipe2(_r.recipe_name, _r.ingredients, _r.tech, _r.config, _r.filters)
	end
end
```
