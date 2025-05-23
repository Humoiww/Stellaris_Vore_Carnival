# Situations System
# -----------------------
#
# The Situations system is used to for tracking and interacting with ongoing stories in your empire.
# A planetary revolt, a starbase slowly falling into a black hole, or an empire-wide resource deficit are all examples of possible situations.
#
# Situations always have:
# 	- A TARGET (typically a single planet, your whole empire, or no target)
# 	- A PROGRESS BAR that goes from 0 to 100, split in one or more STAGES
# 	- Different APPROACHES used to define how to interact with the ongoing event
# 	- Monthly EVENTS, both random and fixed
# 	- One or two ENDINGS triggering when the progress bar reaches 0 and/or 100
#
# This complex, flexible system allows you to craft a huge variety of situations.
# You could have a situation starting from 100 where you are trying to *reduce* progress, for example,
# or one where you start from the middle and have to choose between two contrasting goals.
# The system is has lots of moving parts, so let's examine them one at a time.
#
#
# BASIC SETUP
# -----------------------
# All scopes are this/root = situation. You can scope to owner for the country or target for the target.
# Situations typically target a planet or a country. If no target is set, the UI will indicate your empire (but "target = { }" in script will not work).
# Targeting other scopes, while possible, breaks triggered target modifiers, so target with caution.
# AI empires can also be target of Situations. This can be useful to simulate complex behaviours.
# Mercenary Enclaves in Overlord, for example, use a situation players will never see to handle patron rewards.
#
# test_situation = {					#For loc strings, you need to define "test_situation", "test_situation_type", "test_situation_desc", and "test_situation_monthly_change_tooltip"
# 										#first is the name while the Situation exists. It can use [Target.GetName] and so on),
# 										#The _type key is used in cases where the Situation does not exist yet,
# 										#specifically the start_situation effect's tooltip (stuff like [Target.GetName] will not work there.)
# 										#The monthly_change_tooltip describes what the player can do to affect the situation's progress.
#										#It can be seen when tooltipping over the monthly change value.
#
# 	desc = { text = x triggers = { } }	#default is <key>_desc, but you can do triggered overrides
# 	picture = GFX_evt_alien_nature 		#Also supports triggered pictures like in events
# 	category = positive/negative/neutral #Affects the tone of various UI elements
# }
#
# Situations can have up to two endings - one reached when progress reaches o (on_fail), the other when progress reaches its completion (on_progress_complete)
#
# 	complete_icon = GFX_icon			#Defines icon that will appear on the right side of the bar
# 	complete_icon_frame = GFX_icon_frame
# 	fail_icon = GFX_icon				#Defines icon that will appear on the left side of the bar
# 	fail_icon_frame = GFX_icon_frame
#
# 	on_start = { }						#Effects when the Situation is created
# 	on_progress_complete = {			#Effects when Situation's progress has reached its completion
# 										#You should always call destroy_situation = this from here or from an event fired by here
# 	}
# 	on_fail = {							#Effects when Situation's total progress is below 0
# 										#You should always call destroy_situation = this from here or from an event fired by here
# 	}
# 	on_abort = {}						#Effects when Situation is cancelled via abort_trigger trigger
# 	abort_trigger = {} 					#Trigger for when a Situation should abort, firing on_abort
#
# Situations can have modifiers. They will be applied as long as the Situation is active.
#
# 	modifier = { } 						#Modifier applying to the country experiencing the situation
# 	triggered_modifier = {				#Triggered modifier applying to the country experiencing the situation, if the triggers are true
# 		potential = { }					#The triggers to check for the modifier to be applied
# 		modifier = { }					#The modifier being applied (see modifiers.log for a complete list)
# 	}
#
# 	target_modifier = { } 				#Modifier applying to the target planet of a Situation. Does not work on other scope types!
# 	triggered_target_modifier = {		#Modifier applying to the target planet of a Situation, if the triggers are true. Does not work on other scope types!
# 		potential = { }
# 		modifier = { }
# 	}
#
#   This is a scriptable alert for when a situation has been blocked and requires an action from the player. If a trigger passes for any situation it will display the alert.
#   There can be multiple descriptions for multiple situations at a time, everything will be displayed in the tooltip of the alert.
#   triggered_blocked_desc = {
#       trigger = {}
#       text = ""
#   }
#
#
# PROGRESS BAR
# -----------------------
# Each Situation has a progress bar ranging from 1 to the completion value (generated by the end of the final stage).
# Progress can start anywhere in the bar.
#
# 	start_value = 20					# This is the minimum value that the situation can have (so the progress is contained between `start_value` and the `end` of the last stage )
#	initial_progress = 50				# The value the situation progress is going to have when starting. Important for bidirectional situations.
										# if it is lower than start_value, it will be changed to start_value
# 	progress_direction = bidirectional	#monodirectional/bidirectional (defaults to monodirectional)
# 	complete_category = positive		#Only for progress_direction = bidirectional - only affects progress towards completetion (right)
# 	fail_category = negative			#Only for progress_direction = bidirectional - only affects progress towards fail (left)
#
# The progress bar is divided in one or more stages. Each stage can have different modifiers associated with it.
#
# stages = { 							#List all your stages here, in the correct order. You need at least one.
#
# 	sample_stage = { 					#Expects the key to be localised. You can also define <key>_desc and it will show on the tooltip, but this is optional.
# 		icon = GFX_asset_name
# 		background = GFX_asset_name
# 		end = 40						# Where the stage ends and a new one starts. Determines the end value of the Situation, if it is the final stage.
# 		on_first_enter = { }			#Effect for the first time this stage fires
# 		modifier/triggered_modifier/target_modifier/triggered_target_modifier = { } #as standard (applies only during this Stage)
# 		custom_tooltip = x 				#this will print in the stage tooltip (on_first_enter will not, because spoilers; modifiers will)
# 	}
# }
#
# Every month, progress ticks up or down based on factors defined in monthly_progress:
#
# monthly_progress = {
# 	base = 1							# default monthly progress
# 	modifier = { 						#Standard weight fields, you can do everything that common/script_values/00_script_values.txt tells you about
# 		add = 2
# 		desc = federation_acceptance_reduce_fleet 	#Do not forget to add loc tooltips (the game will scream at you otherwise)
# 		<triggers>
# 	}
# }
#
#
# APPROACHES
# -----------------------
# Situations approaches allow players to define how they're dealing with the ongoing situation.
# Approaches can have different triggers, upkeep costs and modifiers associated with them.
#
# approach = {
# 	name = approach_a 					#This is localised. If you add a loc key "approach_a_desc", it will also show, but this is optional.
# 	icon = GFX_asset_icon
# 	icon_background = GFX_asset_icon_background
# 	allow = { } 						#If this fails, the Approach is greyed out
# 	potential = { } 					#If this fails, the Approach is not shown
# 	on_select = { } 					#Effect when you pick the Approach.
# 	default = yes 						#Sets the Approach to be the Situation's default. This means that it will be picked when the Situation starts, and if the current
# 										#Approach is invalidated (fails potential or allow check). This won't happen while the Situation is locked, so events which demand
# 										#a choice of Approach can lock the situation (set_situation_locked) in immediate (or the Situation's on_start) and unlock them in after.
#
# 	modifier/triggered_modifier/target_modifier/triggered_target_modifier = { } #as standard (applies only when this Approach is picked)
#
# 	resources = { 						#Resource table as standard
# 		category = situations
# 		cost = { }
# 		upkeep = { }
# 		produces = { }
# 	}
# 	ai_weight = { } 					#AI will pick the one with the highest weight
# }
#
#
# EVENTS
# -----------------------
# Situations have their own own_action used to set either random or recurring events.
#
# 	on_monthly = { #Note: technically, this is an on_action called test_situation. So don't call your situation "on_monthly_pulse"!
# 		events = {
# 	    }
# 		random_events = {
# 	    }
# 	}
#
#  IMPORTANT NOTE: events impacting the situations itself (by adding progress, for example) should have their effects set
#  in an immediate = {} block - lest the players leave them unopened, but unclicked to avoid their negative outcomes.
#  Effects in immediate = {} don't always appear correctly in tooltips, though! Test your events carefully and, if necessary
#  use tooltip = {} in event options to correctly generate the tooltip.
#
#
#  EXAMPLE
#  -----------------------
#  All scopes are this/root = situation. You can scope to owner for the country or target for the target.
#
#  test_situation = {					#For loc strings, you need to define "test_situation", "test_situation_type", and "test_situation_desc"
# 										#first is the name while the Situation exists iIt can use [Target.GetName] and so on),
# 										#The _type key is used in cases where the Situation does not exist yet,
# 										#specifically the start_situation effect's tooltip (stuff like [Target.GetName] will not work there.)
# 	desc = { text = x triggers = { } }	#default is <key>_desc, but you can do triggered overrides
# 	picture = GFX_evt_alien_nature 		#Also supports triggered pictures like in events
# 	category = positive/negative/neutral #Affects the tone of various UI elements
#
# 	# Endings
# 	complete_icon = GFX_icon			#Defines icon that will appear on the right side of the bar
# 	complete_icon_frame = GFX_icon_frame
# 	fail_icon = GFX_icon				#Defines icon that will appear on the left side of the bar
# 	fail_icon_frame = GFX_icon_frame
#
# 	on_start = { }						#Effects when the Situation is created
# 	on_progress_complete = {			#Effects when Situation's progress has reached completion
# 										#You should always call destroy_situation = this from here or from an event fired by here
# 	}
# 	on_fail = {							#Effects when Situation's total progress is below 0
# 										#You should always call destroy_situation = this from here or from an event fired by here
# 	}
# 	on_abort = {}						#Effects when Situation is cancelled via abort_trigger trigger
# 	abort_trigger = {} 					#Trigger for when a Situation should abort, firing on_abort
#
# 	# Modifiers
# 	modifier = { } 						#Modifier applying to the country experiencing the situation
# 	triggered_modifier = {				#Triggered modifier applying to the country experiencing the situation, if the triggers are true
# 		potential = { }
# 		modifier = { }
# 	}
#
# 	target_modifier = { } 				#Modifier applying to the target planet of a Situation. Does not work on other scope types!
# 	triggered_target_modifier = {		#Modifier applying to the target planet of a Situation, if the triggers are true. Does not work on other scope types!
# 		potential = { }
# 		modifier = { }
# 	}
#
# 	#Progress
# 	start_value = 20					#Situation will start at this number. Default is 0
# 	progress_direction = bidirectional	#monodirectional/bidirectional (defaults to monodirectional)
# 	complete_category = positive		#Only for progress_direction = bidirectional - only affects progress towards completetion (right)
# 	fail_category = negative			#Only for progress_direction = bidirectional - only affects progress towards fail (left)
#
# 	stages = { 							#List all your stages here, in the correct order.
#
# 		sample_stage = { 					#Expects the key to be localised. You can also define <key>_desc and it will show on the tooltip, but this is optional.
# 			icon = GFX_asset_name
# 			background = GFX_asset_name
# 			end = 40						# Where the stage ends and a new one starts. Determines the end value of the Situation, if it is the final stage.
# 			on_first_enter = { }			#Effect for the first time this stage fires
# 			modifier/triggered_modifier/target_modifier/triggered_target_modifier = { } #as standard (applies only during this Stage)
# 			custom_tooltip = x 				#this will print in the stage tooltip (on_first_enter will not, because spoilers; modifiers will)
# 		}
# 	}
#
# 	monthly_progress = {
# 		base = 1							# default monthly progress
# 		modifier = { 						#Standard weight fields, you can do everything that common/script_values/00_script_values.txt tells you about
# 			add = 2
# 			desc = federation_acceptance_reduce_fleet 	#Do not forget to add loc tooltips (the game will scream at you otherwise)
# 			<triggers>
# 		}
# 	}
#
# 	#Approaches
# 	approach = {
# 		name = approach_a 					#This is localised. If you add a loc key "approach_a_desc", it will also show, but this is optional.
# 		icon = GFX_asset_icon
# 		icon_background = GFX_asset_icon_background
# 		allow = { } 						#If this fails, the Approach is greyed out
# 		potential = { } 					#If this fails, the Approach is not shown
# 		on_select = { } 					#Effect when you pick the Approach.
#
# 		default = yes 						#Sets the Approach to be the Situation's default. This means that it will be picked when the Situation starts, and if the current
# 											#Approach is invalidated (fails potential or allow check). This won't happen while the Situation is locked, so events which demand
# 											#a choice of Approach can lock the situation (set_situation_locked) in immediate (or the Situation's on_start) and unlock them in after.
#
# 		modifier/triggered_modifier/target_modifier/triggered_target_modifier = { } #as standard (applies only when this Approach is picked)
#
# 		resources = { 						#Resource table as standard
# 			category = situations
# 			cost = { }
# 			upkeep = { }
# 			produces = { }
# 		}
# 		ai_weight = { } 					#AI will pick the one with the highest weight
# 	}
#
# 	# Events
# 	on_monthly = { #Note: technically, this is an on_action called test_situation. So don't call your situation "on_monthly_pulse"!
# 		events = {
# 		}
# 		random_events = {
# 		}
# 	}
#  }
