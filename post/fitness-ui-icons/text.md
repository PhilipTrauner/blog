# Obtaining Apple Health Workout Icons

<style>
/* https://github.com/Python-Markdown/markdown/issues/845 */
.markdown-body li > p {
	margin-top: 0px;
}
</style>

I've recently been playing around with `SwiftUI` and `HealthKit`.
Naturally, the first thing I wanted to try out was to display a list of all workouts and their respective types.

Can't be that difficult, right?


## 🧪 Discovery

SF Symbols contains _some_ health-related icons, but none that depict workout types (except walking).
The `HealthKit` documentation also doesn't mention anything in regards to workout icons or even localized names.

After a bit of searching I stumbled across references to `FitnessUI.framework`, which I assumed to be responsible for presentational aspects of Apple's own `HealthKit` apps.

This was coincidentally also the first time I found out about private frameworks, which one can theoretically use, but only if violating App Store guidelines isn't a concern.
`FitnessUI` is one such framework, but because there's at least [one](https://apps.apple.com/us/app/healthfit/id1202650514) app that not only uses the official icons, but also displays them on its store page, I figured that icons might be an accepted gray-area.

`FitnessUI.framework` is not included in macOS builds, therefor I took apart an iOS `.ipsw` to extract it from the contained root filesystem.
I discovered an `Assets.car` file contained within the framework, which I managed to extract with [@_inside](https://twitter.com/_inside)'s [Asset Catalog Tinkerer](https://github.com/insidegui/AssetCatalogTinkerer).

<p align="center"><img src="asset-catalog.png" width="500px"/></p>
<figcaption>Asset Catalog Tinkerer</figcaption>

After extracting all icons it became apparent that there were a few outliers in Apple's naming scheme.

```json
[
    "_112_Normal@3x.png",
    "_112px-1_Normal@3x.png",
    "_112px_Normal@3x.png",
]
```
<figcaption>Suffixes of icons</figcaption>

Some workout types also had different names than their [`HKWorkoutActivityType`](https://developer.apple.com/documentation/healthkit/hkworkoutactivitytype) counterparts.

```json
{
    "australianFootball": "australian_rules_football",
    "cardioDance": "dance",
    "cycling": "outdoorcycle",
    "danceInspiredTraining": "dance_insp_training",
    "discSports": "disk_sports",
    "fitnessGaming": "gaming_sports",
    "functionalStrengthTraining": "func_strength_training",
    "highIntensityIntervalTraining": "hiit",
    "mixedCardio": "mixed_meta_cardio_training",
    "mixedMetabolicCardioTraining": "mixed_meta_cardio_training",
    "preparationAndRecovery": "prep_and_recovery",
    "running": "outdoorrun",
    "socialDance": "social-dance",
    "stairClimbing": "stairs",
    "surfingSports": "surfing",
    "swimming": "swimopen",
    "traditionalStrengthTraining": "trad_weight_training",
    "walking": "outdoorwalk",
    "wheelchairRunPace": "wheelchairrun",
    "wheelchairWalkPace": "wheelchairwalk",
}
```
<figcaption>Rewrite rules for icon names</figcaption>


There are quite a few workout types, and new once are introduced frequently, which is why I ended up automating the process of generating an [`HKWorkoutActivityType`](https://developer.apple.com/documentation/healthkit/hkworkoutactivitytype) extension that correlates workout types and their respective icons.

## ⚙️ Process


1. [Download](https://ipsw.me/) a recent iOS update `.ipsw`
2. Change its file extension to `.zip` and consequently extract it
	<p align="center"><img src="rename-to-zip.png" width="500px"/></p>
3. Open the largest contained `.dmg`
	<p align="center"><img src="largest-dmg.png" width="500px"/></p>
4. Navigate to `System/Library/PrivateFrameworks/FitnessUI.framework`
	<p align="center"><img src="selected-asset-catalog.png" width="500px"/></p>
5. Extract `Assets.car`
6. Open extracted asset catalog with [Asset Catalog Tinkerer](https://github.com/insidegui/AssetCatalogTinkerer)
7. Profit!

## 🪜 Usage
That covers the extraction aspect, but one should ideally be able to correlate the icons with their respective [`HKWorkoutActivityType`](https://developer.apple.com/documentation/healthkit/hkworkoutactivitytype) variants.


```swift
import Foundation
import HealthKit

extension HKWorkoutActivityType {
    var name: String {
        switch self {
        case .americanFootball: return "American Football"
        case .archery: return "Archery"
        case .australianFootball: return "Australian Football"
        case .badminton: return "Badminton"
        case .barre: return "Barre"
        case .baseball: return "Baseball"
        case .basketball: return "Basketball"
        case .bowling: return "Bowling"
        case .boxing: return "Boxing"
        case .cardioDance: return "Cardio Dance"
        case .climbing: return "Climbing"
        case .cooldown: return "Cooldown"
        case .coreTraining: return "Core Training"
        case .cricket: return "Cricket"
        case .crossCountrySkiing: return "Cross Country Skiing"
        case .crossTraining: return "Cross Training"
        case .curling: return "Curling"
        case .cycling: return "Cycling"
        case .dance: return "Dance"
        case .danceInspiredTraining: return "Dance Inspired Training"
        case .discSports: return "Disc Sports"
        case .downhillSkiing: return "Downhill Skiing"
        case .elliptical: return "Elliptical"
        case .equestrianSports: return "Equestrian Sports"
        case .fencing: return "Fencing"
        case .fishing: return "Fishing"
        case .fitnessGaming: return "Fitness Gaming"
        case .flexibility: return "Flexibility"
        case .functionalStrengthTraining: return "Functional Strength Training"
        case .golf: return "Golf"
        case .gymnastics: return "Gymnastics"
        case .handCycling: return "Hand Cycling"
        case .handball: return "Handball"
        case .highIntensityIntervalTraining: return "High Intensity Interval Training"
        case .hiking: return "Hiking"
        case .hockey: return "Hockey"
        case .hunting: return "Hunting"
        case .jumpRope: return "Jump Rope"
        case .kickboxing: return "Kickboxing"
        case .lacrosse: return "Lacrosse"
        case .martialArts: return "Martial Arts"
        case .mindAndBody: return "Mind and Body"
        case .mixedCardio: return "Mixed Cardio"
        case .mixedMetabolicCardioTraining: return "Mixed Metabolic Cardio Training"
        case .other: return "Other"
        case .paddleSports: return "Paddle Sports"
        case .pickleball: return "Pickleball"
        case .pilates: return "Pilates"
        case .play: return "Play"
        case .preparationAndRecovery: return "Preparation and Recovery"
        case .racquetball: return "Racquetball"
        case .rowing: return "Rowing"
        case .rugby: return "Rugby"
        case .running: return "Running"
        case .sailing: return "Sailing"
        case .skatingSports: return "Skating Sports"
        case .snowSports: return "Snow Sports"
        case .snowboarding: return "Snowboarding"
        case .soccer: return "Soccer"
        case .socialDance: return "Social Dance"
        case .softball: return "Softball"
        case .squash: return "Squash"
        case .stairClimbing: return "Stair Climbing"
        case .stairs: return "Stairs"
        case .stepTraining: return "Step Training"
        case .surfingSports: return "Surfing Sports"
        case .swimming: return "Swimming"
        case .tableTennis: return "Table Tennis"
        case .taiChi: return "Tai Chi"
        case .tennis: return "Tennis"
        case .trackAndField: return "Track and Field"
        case .traditionalStrengthTraining: return "Traditional Strength Training"
        case .volleyball: return "Volleyball"
        case .walking: return "Walking"
        case .waterFitness: return "Water Fitness"
        case .waterPolo: return "Water Polo"
        case .waterSports: return "Water Sports"
        case .wheelchairRunPace: return "Wheelchair Run Pace"
        case .wheelchairWalkPace: return "Wheelchair Walk Pace"
        case .wrestling: return "Wrestling"
        case .yoga: return "Yoga"
        default: return "Unknown"
        }
    }

    static let allCases: [HKWorkoutActivityType] = [
        .americanFootball,
        .archery,
        .australianFootball,
        .badminton,
        .barre,
        .baseball,
        .basketball,
        .bowling,
        .boxing,
        .cardioDance,
        .climbing,
        .cooldown,
        .coreTraining,
        .cricket,
        .crossCountrySkiing,
        .crossTraining,
        .curling,
        .cycling,
        .dance,
        .danceInspiredTraining,
        .discSports,
        .downhillSkiing,
        .elliptical,
        .equestrianSports,
        .fencing,
        .fishing,
        .fitnessGaming,
        .flexibility,
        .functionalStrengthTraining,
        .golf,
        .gymnastics,
        .handCycling,
        .handball,
        .highIntensityIntervalTraining,
        .hiking,
        .hockey,
        .hunting,
        .jumpRope,
        .kickboxing,
        .lacrosse,
        .martialArts,
        .mindAndBody,
        .mixedCardio,
        .mixedMetabolicCardioTraining,
        .other,
        .paddleSports,
        .pickleball,
        .pilates,
        .play,
        .preparationAndRecovery,
        .racquetball,
        .rowing,
        .rugby,
        .running,
        .sailing,
        .skatingSports,
        .snowSports,
        .snowboarding,
        .soccer,
        .socialDance,
        .softball,
        .squash,
        .stairClimbing,
        .stairs,
        .stepTraining,
        .surfingSports,
        .swimming,
        .tableTennis,
        .taiChi,
        .tennis,
        .trackAndField,
        .traditionalStrengthTraining,
        .volleyball,
        .walking,
        .waterFitness,
        .waterPolo,
        .waterSports,
        .wheelchairRunPace,
        .wheelchairWalkPace,
        .wrestling,
        .yoga
    ]

    var url: URL? {
        switch self {
        case .americanFootball: return Bundle.main.url(forResource: "american_football_112px_Normal@3x", withExtension: "png")
        case .archery: return Bundle.main.url(forResource: "archery_112px_Normal@3x", withExtension: "png")
        case .australianFootball: return Bundle.main.url(forResource: "australian_rules_football_112px_Normal@3x", withExtension: "png")
        case .badminton: return Bundle.main.url(forResource: "badminton_112px_Normal@3x", withExtension: "png")
        case .barre: return Bundle.main.url(forResource: "barre_112px_Normal@3x", withExtension: "png")
        case .baseball: return Bundle.main.url(forResource: "baseball_112px_Normal@3x", withExtension: "png")
        case .basketball: return Bundle.main.url(forResource: "basketball_112px_Normal@3x", withExtension: "png")
        case .bowling: return Bundle.main.url(forResource: "bowling_112px_Normal@3x", withExtension: "png")
        case .boxing: return Bundle.main.url(forResource: "boxing_112px_Normal@3x", withExtension: "png")
        case .cardioDance: return Bundle.main.url(forResource: "dance_112px_Normal@3x", withExtension: "png")
        case .climbing: return Bundle.main.url(forResource: "climbing_112px_Normal@3x", withExtension: "png")
        case .cooldown: return Bundle.main.url(forResource: "cooldown_112px_Normal@3x", withExtension: "png")
        case .coreTraining: return Bundle.main.url(forResource: "core_training_112px-1_Normal@3x", withExtension: "png")
        case .cricket: return Bundle.main.url(forResource: "cricket_112px_Normal@3x", withExtension: "png")
        case .crossCountrySkiing: return Bundle.main.url(forResource: "cross_country_skiing_112px_Normal@3x", withExtension: "png")
        case .crossTraining: return Bundle.main.url(forResource: "cross_training_112px_Normal@3x", withExtension: "png")
        case .curling: return Bundle.main.url(forResource: "curling_112px_Normal@3x", withExtension: "png")
        case .cycling: return Bundle.main.url(forResource: "outdoorcycle_112px_Normal@3x", withExtension: "png")
        case .dance: return Bundle.main.url(forResource: "dance_112px_Normal@3x", withExtension: "png")
        case .danceInspiredTraining: return Bundle.main.url(forResource: "dance_insp_training_112px_Normal@3x", withExtension: "png")
        case .discSports: return Bundle.main.url(forResource: "disk_sports_112px-1_Normal@3x", withExtension: "png")
        case .downhillSkiing: return Bundle.main.url(forResource: "downhill_skiing_112px_Normal@3x", withExtension: "png")
        case .elliptical: return Bundle.main.url(forResource: "elliptical_112px_Normal@3x", withExtension: "png")
        case .equestrianSports: return Bundle.main.url(forResource: "equestrian_sports_112px_Normal@3x", withExtension: "png")
        case .fencing: return Bundle.main.url(forResource: "fencing_112px_Normal@3x", withExtension: "png")
        case .fishing: return Bundle.main.url(forResource: "fishing_112px_Normal@3x", withExtension: "png")
        case .fitnessGaming: return Bundle.main.url(forResource: "gaming_sports_112px-1_Normal@3x", withExtension: "png")
        case .flexibility: return Bundle.main.url(forResource: "flexibility_112px_Normal@3x", withExtension: "png")
        case .functionalStrengthTraining: return Bundle.main.url(forResource: "func_strength_training_112_Normal@3x", withExtension: "png")
        case .golf: return Bundle.main.url(forResource: "golf_112px_Normal@3x", withExtension: "png")
        case .gymnastics: return Bundle.main.url(forResource: "gymnastics_112px_Normal@3x", withExtension: "png")
        case .handCycling: return Bundle.main.url(forResource: "hand_cycling_112px_Normal@3x", withExtension: "png")
        case .handball: return Bundle.main.url(forResource: "handball_112px_Normal@3x", withExtension: "png")
        case .highIntensityIntervalTraining: return Bundle.main.url(forResource: "hiit_112px_Normal@3x", withExtension: "png")
        case .hiking: return Bundle.main.url(forResource: "hiking_112px_Normal@3x", withExtension: "png")
        case .hockey: return Bundle.main.url(forResource: "hockey_112px_Normal@3x", withExtension: "png")
        case .hunting: return Bundle.main.url(forResource: "hunting_112px_Normal@3x", withExtension: "png")
        case .jumpRope: return Bundle.main.url(forResource: "jump_rope_112px_Normal@3x", withExtension: "png")
        case .kickboxing: return Bundle.main.url(forResource: "kickboxing_112px_Normal@3x", withExtension: "png")
        case .lacrosse: return Bundle.main.url(forResource: "lacrosse_112px_Normal@3x", withExtension: "png")
        case .martialArts: return Bundle.main.url(forResource: "martial_arts_112px_Normal@3x", withExtension: "png")
        case .mindAndBody: return Bundle.main.url(forResource: "mind_and_body_112px_Normal@3x", withExtension: "png")
        case .mixedCardio: return Bundle.main.url(forResource: "mixed_meta_cardio_training_112px_Normal@3x", withExtension: "png")
        case .mixedMetabolicCardioTraining: return Bundle.main.url(forResource: "mixed_meta_cardio_training_112px_Normal@3x", withExtension: "png")
        case .other: return Bundle.main.url(forResource: "other_112px_Normal@3x", withExtension: "png")
        case .paddleSports: return Bundle.main.url(forResource: "paddle_sports_112px_Normal@3x", withExtension: "png")
        case .pickleball: return Bundle.main.url(forResource: "pickleball_112px_Normal@3x", withExtension: "png")
        case .pilates: return Bundle.main.url(forResource: "pilates_112px_Normal@3x", withExtension: "png")
        case .play: return Bundle.main.url(forResource: "play_112px_Normal@3x", withExtension: "png")
        case .preparationAndRecovery: return Bundle.main.url(forResource: "prep_and_recovery_112px_Normal@3x", withExtension: "png")
        case .racquetball: return Bundle.main.url(forResource: "racquetball_112px_Normal@3x", withExtension: "png")
        case .rowing: return Bundle.main.url(forResource: "rowing_112px_Normal@3x", withExtension: "png")
        case .rugby: return Bundle.main.url(forResource: "rugby_112px_Normal@3x", withExtension: "png")
        case .running: return Bundle.main.url(forResource: "outdoorrun_112px_Normal@3x", withExtension: "png")
        case .sailing: return Bundle.main.url(forResource: "sailing_112px_Normal@3x", withExtension: "png")
        case .skatingSports: return Bundle.main.url(forResource: "skating_sports_112px_Normal@3x", withExtension: "png")
        case .snowSports: return Bundle.main.url(forResource: "snow_sports_112px_Normal@3x", withExtension: "png")
        case .snowboarding: return Bundle.main.url(forResource: "snowboarding_112px_Normal@3x", withExtension: "png")
        case .soccer: return Bundle.main.url(forResource: "soccer_112px_Normal@3x", withExtension: "png")
        case .socialDance: return Bundle.main.url(forResource: "social-dance_112px-1_Normal@3x", withExtension: "png")
        case .softball: return Bundle.main.url(forResource: "softball_112px_Normal@3x", withExtension: "png")
        case .squash: return Bundle.main.url(forResource: "squash_112px_Normal@3x", withExtension: "png")
        case .stairClimbing: return Bundle.main.url(forResource: "stairs_112px_Normal@3x", withExtension: "png")
        case .stairs: return Bundle.main.url(forResource: "stairs_112px_Normal@3x", withExtension: "png")
        case .stepTraining: return Bundle.main.url(forResource: "step_training_112px_Normal@3x", withExtension: "png")
        case .surfingSports: return Bundle.main.url(forResource: "surfing_112px_Normal@3x", withExtension: "png")
        case .swimming: return Bundle.main.url(forResource: "swimopen_112px_Normal@3x", withExtension: "png")
        case .tableTennis: return Bundle.main.url(forResource: "table_tennis_112px_Normal@3x", withExtension: "png")
        case .taiChi: return Bundle.main.url(forResource: "tai_chi_112px_Normal@3x", withExtension: "png")
        case .tennis: return Bundle.main.url(forResource: "tennis_112px_Normal@3x", withExtension: "png")
        case .trackAndField: return Bundle.main.url(forResource: "track_and_field_112px_Normal@3x", withExtension: "png")
        case .traditionalStrengthTraining: return Bundle.main.url(forResource: "trad_weight_training_112px_Normal@3x", withExtension: "png")
        case .volleyball: return Bundle.main.url(forResource: "volleyball_112px_Normal@3x", withExtension: "png")
        case .walking: return Bundle.main.url(forResource: "outdoorwalk_112px_Normal@3x", withExtension: "png")
        case .waterFitness: return Bundle.main.url(forResource: "water_fitness_112px_Normal@3x", withExtension: "png")
        case .waterPolo: return Bundle.main.url(forResource: "water_polo_112px_Normal@3x", withExtension: "png")
        case .waterSports: return Bundle.main.url(forResource: "water_sports_112px_Normal@3x", withExtension: "png")
        case .wheelchairRunPace: return Bundle.main.url(forResource: "wheelchairrun_112px_Normal@3x", withExtension: "png")
        case .wheelchairWalkPace: return Bundle.main.url(forResource: "wheelchairwalk_112px_Normal@3x", withExtension: "png")
        case .wrestling: return Bundle.main.url(forResource: "wrestling_112px_Normal@3x", withExtension: "png")
        case .yoga: return Bundle.main.url(forResource: "yoga_112px_Normal@3x", withExtension: "png")
        default: return nil
        }
    }
}
```
<figcaption><code>HKWorkoutActivityType.swift</code></figcaption>

<details><summary>Script used to generate the extension</summary>

<ol>
	<li>
	<p>Create following folder structure</p>
	<pre>
├── script.py
├── assets
│   ├── all
│   │   └── FitnessUI
└── pyproject.toml</pre>
	</li>
	<li>Copy extracted assets into <code>assets/all/FitnessUI</code></li>
	<li>Run <code>script.py</code></li>
</ol>

<p>All icons that have an associated <a href="https://developer.apple.com/documentation/healthkit/hkworkoutactivitytype"><code>HKWorkoutActivityType</code></a> variant are copied over to <code>assets/reduced/FitnessUI</code>, and the generated code is printed to <code>stdout</code>.</p>

```python
from dataclasses import dataclass
from itertools import chain
from pathlib import Path
from string import Template
from typing import List
from typing import Optional
from shutil import copy

from httpx import get
from inflection import titleize
from inflection import underscore
from jmespath import search


def removesuffixes(text: str, suffixes: List[str]) -> str:
    for suffix in suffixes:
        applied = text.removesuffix(suffix)
        if applied != text:
            return applied
    return text


HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_TEMPLATE = Template(
    """
import Foundation
import HealthKit

extension HKWorkoutActivityType {
    var name: String {
        switch self {
$name
        default: return "Unknown"
        }
    }

    static let allCases: [HKWorkoutActivityType] = [
$all_cases
    ]

    var url: URL? {
        switch self {
$url
        default: return nil
        }
    }
}
""".lstrip()
)

HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_NAME_TEMPLATE = Template(
    '        case .$case: return "$humanized"'
)
HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_ALL_CASES_TEMPLATE = Template("        .$case")
HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_URL_TEMPLATE = Template(
    '        case .$case: return Bundle.main.url(forResource: "$resource", withExtension: "$extension")'
)


HK_WORKOUT_ACTIVITY_TYPE = "https://developer.apple.com/tutorials/data/documentation/healthkit/hkworkoutactivitytype.json"
WORKOUT_ASSET_SUFFIXES = [
    "_112_Normal@3x.png",
    "_112px-1_Normal@3x.png",
    "_112px_Normal@3x.png",
]
WORKOUT_ASSETS_BASE_PATH = Path("assets/all/FitnessUI/")
WORKOUT_ASSETS_OUTPUT_PATH = Path("assets/reduced/FitnessUI/")
WORKOUT_ASSETS = chain(
    *[
        WORKOUT_ASSETS_BASE_PATH.glob(f"*{pattern}")
        for pattern in WORKOUT_ASSET_SUFFIXES
    ]
)
WORKOUT_ASSET_LOOKUP = {
    removesuffixes(asset.name, WORKOUT_ASSET_SUFFIXES): asset
    for asset in WORKOUT_ASSETS
}
WORKOUT_SPECIAL_CASES = {
    "australianFootball": "australian_rules_football",
    "cardioDance": "dance",
    "cycling": "outdoorcycle",
    "danceInspiredTraining": "dance_insp_training",
    "discSports": "disk_sports",
    "fitnessGaming": "gaming_sports",
    "functionalStrengthTraining": "func_strength_training",
    "highIntensityIntervalTraining": "hiit",
    "mixedCardio": "mixed_meta_cardio_training",
    "mixedMetabolicCardioTraining": "mixed_meta_cardio_training",
    "preparationAndRecovery": "prep_and_recovery",
    "running": "outdoorrun",
    "socialDance": "social-dance",
    "stairClimbing": "stairs",
    "surfingSports": "surfing",
    "swimming": "swimopen",
    "traditionalStrengthTraining": "trad_weight_training",
    "walking": "outdoorwalk",
    "wheelchairRunPace": "wheelchairrun",
    "wheelchairWalkPace": "wheelchairwalk",
}

@dataclass
class Workout:
    case: str
    asset_path: Optional[Path]

    @property
    def humanized(self) -> str:
        return Workout.humanize_by_case(self.case)

    @staticmethod
    def match_asset_by_case(case: str) -> Optional[Path]:
        case_override = (
            case if case not in WORKOUT_SPECIAL_CASES else WORKOUT_SPECIAL_CASES[case]
        )

        for name in (
            case_override,
            underscore(case_override),
            underscore(case_override).replace("_", ""),
        ):
            if name in WORKOUT_ASSET_LOOKUP:
                return WORKOUT_ASSET_LOOKUP[name]

    @staticmethod
    def humanize_by_case(case: str) -> str:
        return titleize(underscore(case)).replace("And", "and")

    @staticmethod
    def from_enum_case(case: str):
        return Workout(case, Workout.match_asset_by_case(case))

    @property
    def template_mapping(self):
        return {
            "case": self.case,
            "humanized": self.humanized,
            "resource": self.asset_path.stem,
            "extension": self.asset_path.suffix.removeprefix("."),
        }


r = get(HK_WORKOUT_ACTIVITY_TYPE)
json = r.json()


# Only include non-deprecated workouts:
# references.* | [?kind==`symbol`&&deprecated==null].fragments[?[0].text==`case `][1].text
workouts_variants = search(
    "references.* | [?kind==`symbol`].fragments[?[0].text==`case `][1].text", json
)
workouts = sorted(
    [Workout.from_enum_case(workout) for workout in workouts_variants],
    key=lambda workout: workout.case,
)

# Only copy resources that are referenced by workouts
for workout in workouts:
    asset_path = workout.asset_path
    if asset_path is not None:
        copy(asset_path, WORKOUT_ASSETS_OUTPUT_PATH)

print(
    HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_TEMPLATE.safe_substitute(
        name="\n".join(
            (
                HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_NAME_TEMPLATE.substitute(
                    workout.template_mapping
                )
                for workout in workouts
            )
        ),
        all_cases=",\n".join(
            (
                HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_ALL_CASES_TEMPLATE.substitute(
                    workout.template_mapping
                )
                for workout in workouts
            )
        ),
        url="\n".join(
            (
                HK_WORKOUT_ACTIVITY_TYPE_EXTENSION_URL_TEMPLATE.substitute(
                    workout.template_mapping
                )
                for workout in workouts
            )
        ),
    )
)
```
<figcaption><code>script.py</code></figcaption>

```toml
httpx = "^0.16.1"
jmespath = "^0.10.0"
inflection = "^0.5.1"
```
<figcaption><code>pyproject.toml</code> excerpt</figcaption>

</details>

