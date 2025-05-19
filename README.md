## 👥 Group Members

| Name                | Student ID     |
|---------------------|----------------|
| Yabets Zekaryas     | Ugr/25317/14   |
| Elyas  Damenu       | Ugr/25386/14   |
| Mahlet Demeke       | Ugr/25398/14   |
| Meron Tilahun       | Ugr/25700/14   |
| Yididya Tesfaye     | Ugr/25471/14   |

# 📘 Project Description – Recipe Voyager

Recipe Voyager is a privacy-conscious recipe discovery platform that provides users with a fast, structured, and intuitive way to explore global cuisines. Built with performance, accessibility, and privacy in mind, the portal leverages open APIs to fetch meal data while ensuring user anonymity throughout the experience.

🛠️ Technologies Used

    Frontend: React (with TypeScript)
    Routing: React Router
    UI Components: ShadCN UI, Tailwind CSS
    API Integration: TheMealDB REST API
    SEO & Markup: React Helmet, Schema.org JSON-LD

index.tsx
---------

```

  // Fetch initial recipes
  useEffect(() => {
    const fetchInitialRecipes = async () => {
      setLoading(true);
      try {
        // Fetch a few initial recipes (we'll get random recipe and then search for similar ones)
        const randomRecipe = await getRandomRecipe();
        if (randomRecipe?.strCategory) {
          const similarRecipes = await filterByCategory(randomRecipe.strCategory);
          setRecipes([...similarRecipes]);
        } else {
          // Fallback to a common search term
          const defaultRecipes = await searchRecipesByName("chicken");
          setRecipes(defaultRecipes);
        }
      } catch (error) {
        console.error("Error fetching initial recipes:", error);
        toast.error("Failed to load recipes. Please try again later.");
      } finally {
        setLoading(false);
      }
    };

    fetchInitialRecipes();
  }, []);

  const handleSearch = async (query: string) => {
    setSearchQuery(query);
    setActiveFilter(null);
    setLoading(true);
    
    try {
      const results = await searchRecipesByName(query);
      setRecipes(results);
      
      if (results.length === 0) {
        toast.info(`No recipes found for "${query}"`);
      } else {
        recipesRef.current?.scrollIntoView({ behavior: "smooth", block: "start" });
      }
    } catch (error) {
      console.error("Error searching recipes:", error);
      toast.error("Failed to search recipes. Please try again later.");
    } finally {
      setLoading(false);
    }
  };

  const handleCategorySelect = async (category: string) => {
    setSearchQuery("");
    setActiveFilter({ type: "category", value: category });
    setLoading(true);
    
    try {
      const results = await filterByCategory(category);
      setRecipes(results);
      recipesRef.current?.scrollIntoView({ behavior: "smooth", block: "start" });
    } catch (error) {
      console.error("Error filtering by category:", error);
      toast.error("Failed to load recipes. Please try again.");
    } finally {
      setLoading(false);
    }
  };

  const handleAreaSelect = async (area: string) => {
    setSearchQuery("");
    setActiveFilter({ type: "cuisine", value: area });
    setLoading(true);
    
    try {
      const results = await filterByArea(area);
      setRecipes(results);
      recipesRef.current?.scrollIntoView({ behavior: "smooth", block: "start" });
    } catch (error) {
      console.error("Error filtering by area:", error);
      toast.error("Failed to load recipes. Please try again.");
    } finally {
      setLoading(false);
    }
  };

  const handleExplore = () => {
    // Scroll to the categories section
    window.scrollTo({ 
      top: window.innerHeight - 80, 
      behavior: "smooth" 
    });
  };

  const resetFilters = async () => {
    setSearchQuery("");
    setActiveFilter(null);
    setLoading(true);
    
    try {
      const randomRecipe = await getRandomRecipe();
      if (randomRecipe?.strCategory) {
        const similarRecipes = await filterByCategory(randomRecipe.strCategory);
        setRecipes([...similarRecipes]);
      } else {
        const defaultRecipes = await searchRecipesByName("chicken");
        setRecipes(defaultRecipes);
      }
    } catch (error) {
      console.error("Error resetting filters:", error);
    } finally {
      setLoading(false);
    }
  };
```

This code manages the logic for fetching, searching, and filtering recipes in a Next.js app:

*   On component load, it fetches a random recipe and uses its category to load similar ones, or defaults to chicken recipes.
    
*   It provides functions to:
    
    *   Search recipes by name,
        
    *   Filter by category or cuisine,
        
    *   Reset all filters,
        
    *   And scroll to the explore section.
        
*   It also manages loading states and user notifications via toasts.
    

interaction with the api in recipeService.ts
--------------------------------------------

```

// API documentation: https://www.themealdb.com/api.php

export interface Meal {
  idMeal: string;
  strMeal: string;
  strCategory: string;
  strArea: string;
  strInstructions: string;
  strMealThumb: string;
  strTags: string;
  strYoutube: string;
  ingredients: { ingredient: string; measure: string }[];
}

export interface Category {
  idCategory: string;
  strCategory: string;
  strCategoryThumb: string;
  strCategoryDescription: string;
}

export interface Area {
  strArea: string;
}

const API_BASE_URL = "https://www.themealdb.com/api/json/v1/1";

// Function to format meal data
const formatMealData = (meal: any): Meal => {
  const ingredients: { ingredient: string; measure: string }[] = [];
  
  // TheMealDB API has ingredients stored as strIngredient1, strIngredient2, etc.
  for (let i = 1; i <= 20; i++) {
    const ingredient = meal[`strIngredient${i}`];
    const measure = meal[`strMeasure${i}`];
    
    if (ingredient && ingredient.trim() !== "") {
      ingredients.push({ ingredient, measure });
    }
  }
  
  return {
    idMeal: meal.idMeal,
    strMeal: meal.strMeal,
    strCategory: meal.strCategory,
    strArea: meal.strArea,
    strInstructions: meal.strInstructions,
    strMealThumb: meal.strMealThumb,
    strTags: meal.strTags || "",
    strYoutube: meal.strYoutube || "",
    ingredients
  };
};

// Search recipes by name
export const searchRecipesByName = async (query: string): Promise<Meal[]> => {
  try {
    const response = await fetch(`${API_BASE_URL}/search.php?s=${encodeURIComponent(query)}`);
    const data = await response.json();
    
    if (!data.meals) return [];
    
    return data.meals.map(formatMealData);
  } catch (error) {
    console.error("Error searching recipes:", error);
    return [];
  }
};

// Get recipe by ID
export const getRecipeById = async (id: string): Promise<Meal | null> => {
  try {
    const response = await fetch(`${API_BASE_URL}/lookup.php?i=${id}`);
    const data = await response.json();
    
    if (!data.meals || data.meals.length === 0) return null;
    
    return formatMealData(data.meals[0]);
  } catch (error) {
    console.error("Error fetching recipe:", error);
    return null;
  }
};

// Get all categories
export const getCategories = async (): Promise<Category[]> => {
  try {
    const response = await fetch(`${API_BASE_URL}/categories.php`);
    const data = await response.json();
    
    return data.categories || [];
  } catch (error) {
    console.error("Error fetching categories:", error);
    return [];
  }
};

// Get all areas (cuisines)
export const getAreas = async (): Promise<Area[]> => {
  try {
    const response = await fetch(`${API_BASE_URL}/list.php?a=list`);
    const data = await response.json();
    
    return data.meals || [];
  } catch (error) {
    console.error("Error fetching areas:", error);
    return [];
  }
};

// Filter recipes by category
export const filterByCategory = async (category: string): Promise<Meal[]> => {
  try {
    const response = await fetch(`${API_BASE_URL}/filter.php?c=${encodeURIComponent(category)}`);
    const data = await response.json();
    
    if (!data.meals) return [];
    
    // The filtered results don't have complete info, so we need to fetch each meal
    const detailedMeals: Meal[] = [];
    
    for (const meal of data.meals.slice(0, 10)) { // Limit to 10 results to avoid too many requests
      const detailedMeal = await getRecipeById(meal.idMeal);
      if (detailedMeal) {
        detailedMeals.push(detailedMeal);
      }
    }
    
    return detailedMeals;
  } catch (error) {
    console.error("Error filtering by category:", error);
    return [];
  }
};
```

This code defines **API utility functions and data models** for a recipe app using [TheMealDB API](https://www.themealdb.com/api.php). Here's what happens:

*   **Interfaces** define the structure of meals, categories, and areas for consistent typing.
    
*   `formatMealData` extracts and formats ingredients from raw API meal data.
    
*   The following **API functions** are provided:
    
    *   `searchRecipesByName`: Fetches recipes by name.
        
    *   `getRecipeById`: Fetches a single recipe using its ID.
        
    *   `getCategories`: Retrieves all recipe categories.
        
    *   `getAreas`: Retrieves all available cuisines/regions.
        
    *   `filterByCategory`: Filters recipes by category and fetches full details for the first 10.
        

The code ensures data consistency, handles errors, and limits requests to avoid overloading the API.

```


// Filter recipes by area (cuisine)
export const filterByArea = async (area: string): Promise<Meal[]> => {
  try {
    const response = await fetch(`${API_BASE_URL}/filter.php?a=${encodeURIComponent(area)}`);
    const data = await response.json();
    
    if (!data.meals) return [];
    
    // The filtered results don't have complete info, so we need to fetch each meal
    const detailedMeals: Meal[] = [];
    
    for (const meal of data.meals.slice(0, 10)) { // Limit to 10 results to avoid too many requests
      const detailedMeal = await getRecipeById(meal.idMeal);
      if (detailedMeal) {
        detailedMeals.push(detailedMeal);
      }
    }
    
    return detailedMeals;
  } catch (error) {
    console.error("Error filtering by area:", error);
    return [];
  }
};

// Get random recipe
export const getRandomRecipe = async (): Promise<Meal | null> => {
  try {
    const response = await fetch(`${API_BASE_URL}/random.php`);
    const data = await response.json();
    
    if (!data.meals || data.meals.length === 0) return null;
    
    return formatMealData(data.meals[0]);
  } catch (error) {
    console.error("Error fetching random recipe:", error);
    return null;
  }
};
```

This code adds two API utility functions for the recipe app:

*   **`filterByArea(area)`**:  
    Fetches recipes by cuisine/region. Since results contain only minimal data, it retrieves full details for the first 10 recipes using `getRecipeById`.
    
*   **`getRandomRecipe()`**:  
    Fetches a single random recipe and formats it using `formatMealData`.
    

Both functions handle errors gracefully and ensure consistent data formatting.

while loading show skeleton from recipe.tsx
-------------------------------------------

```

const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const [recipe, setRecipe] = useState<Meal | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchRecipe = async () => {
      if (!id) {
        navigate("/");
        return;
      }

      setLoading(true);
      try {
        const recipeData = await getRecipeById(id);
        
        if (!recipeData) {
          toast.error("Recipe not found");
          navigate("/");
          return;
        }
        
        setRecipe(recipeData);
      } catch (error) {
        console.error("Error fetching recipe:", error);
        toast.error("Failed to load recipe. Please try again later.");
        navigate("/");
      } finally {
        setLoading(false);
      }
    };

    fetchRecipe();
  }, [id, navigate]);

  if (loading) {
    return (
      <div className="recipe-container py-8">
        <Skeleton className="h-10 w-40 mb-2" />
        <Skeleton className="h-8 w-96 mb-8" />
        
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
          <Skeleton className="h-80 rounded-xl" />
          <div>
            <div className="flex space-x-4 mb-6">
              <Skeleton className="h-10 w-32" />
              <Skeleton className="h-10 w-32" />
            </div>
            <Skeleton className="h-6 w-40 mb-4" />
            {Array(6).fill(0).map((_, i) => (
              <Skeleton key={i} className="h-6 w-full mb-3" />
            ))}
          </div>
        </div>
      </div>
    );
  }
after that if the recipes found show them using

  if (!recipe) return null;

  return <RecipeDetail recipe={recipe} />;
};
```

> This Next.js component is responsible for fetching and displaying a recipe based on a URL parameter (id). It handles loading states, error cases, and conditional rendering.

**Step-by-step Breakdown:** Extract id from URL using useParams.
*   Redirect to homepage if id is missing or invalid.
*   Fetch recipe data using getRecipeById(id):
*   If the recipe is found, it’s stored in the state.
*   If not, an error message is shown and the user is redirected.
*   While loading, show skeleton placeholders for a better user experience.
*   If loading is done but no recipe was fetched, render nothing (return null).
*   If recipe is fetched successfully, render the component with the recipe data.

This pattern ensures that users see a smooth loading state, clear error feedback, and only valid content.


on RecipeCard.tsx lazy loading have been added
----------------------------------------------

```

interface RecipeCardProps {
  recipe: Meal;
}

const RecipeCard = ({ recipe }: RecipeCardProps) => {
  return (
    <Link to={`/recipe/${recipe.idMeal}`}>
      <Card className="overflow-hidden transition-all duration-300 hover:shadow-lg hover:-translate-y-1 h-full">
        <div className="aspect-video relative overflow-hidden">
          <img
            src={recipe.strMealThumb}
            alt={recipe.strMeal}
            className="w-full h-full object-cover transition-transform duration-300 hover:scale-105"
            loading="lazy"
            width="400"
            height="225"
          />
          <Badge className="absolute top-2 right-2 bg-recipe-primary hover:bg-recipe-primary/90">
            {recipe.strCategory}
          </Badge>
        </div>
        <CardContent className="pt-4">
          <h3 className="text-lg font-semibold mb-1 line-clamp-1">{recipe.strMeal}</h3>
          <p className="text-sm text-muted-foreground">{recipe.strArea} cuisine</p>
        </CardContent>
        <CardFooter className="flex justify-between items-center pt-0 pb-4">
          <div className="flex items-center gap-1 text-sm text-recipe-secondary font-medium">
            <span>{recipe.ingredients.length} ingredients</span>
          </div>
        </CardFooter>
      </Card>
    </Link>
  );
};
```

This code defines a `RecipeCard` component that displays a clickable recipe preview. It uses **lazy loading** for the image (`loading="lazy"`) to improve performance by delaying image loading until it’s in view. The card shows the recipe’s thumbnail, category, name, cuisine type, and ingredient count, styled with hover effects and responsive layout.

## schema.org markup implementation for RecipeDetail.tsx
```
interface RecipeDetailProps {
  recipe: Meal;
}

const RecipeDetail = ({ recipe }: RecipeDetailProps) => {
  const [activeTab, setActiveTab] = useState<"ingredients" | "instructions">("ingredients");

  const renderInstructions = () => {
    // Split instructions by sentences for better readability
    const sentences = recipe.strInstructions
      .split(/\.\s+/)
      .filter(s => s.trim().length > 0)
      .map(s => s.trim() + (s.endsWith('.') ? '' : '.'));

    return (
      <ol className="space-y-4 list-decimal list-outside ml-5">
        {sentences.map((sentence, index) => (
          <li key={index} className="text-pretty">
            {sentence}
          </li>
        ))}
      </ol>
    );
  };

  // Create schema.org Recipe structured data
  const schemaData = {
    "@context": "https://schema.org",
    "@type": "Recipe",
    "name": recipe.strMeal,
    "image": recipe.strMealThumb,
    "author": {
      "@type": "Organization",
      "name": "Recipe Voyager"
    },
    "description": `${recipe.strMeal} - ${recipe.strArea} cuisine recipe`,
    "recipeCategory": recipe.strCategory,
    "recipeCuisine": recipe.strArea,
    "recipeIngredient": recipe.ingredients.map(item => `${item.measure} ${item.ingredient}`),
    "recipeInstructions": recipe.strInstructions
      .split(/\.\s+/)
      .filter(s => s.trim().length > 0)
      .map((step, index) => ({
        "@type": "HowToStep",
        "position": index + 1,
        "text": step.trim() + (step.endsWith('.') ? '' : '.')
      }))
  };
  return (
    <>
      <Helmet>
        <title>{recipe.strMeal} - Recipe Voyager</title>
        <meta name="description" content={`${recipe.strMeal} - An authentic ${recipe.strArea} ${recipe.strCategory.toLowerCase()} recipe with step-by-step instructions.`} />
        <script type="application/ld+json">
          {JSON.stringify(schemaData)}
        </script>
      </Helmet>
    
      <div className="recipe-container py-8">
        <div className="mb-6">
          <Link to="/">
            <Button variant="ghost" className="mb-4 p-0 hover:bg-transparent">
              <ArrowLeft className="mr-2 h-4 w-4" />
              Back to recipes
            </Button>
          </Link>
          <h1 className="text-3xl md:text-4xl font-bold mb-2">{recipe.strMeal}</h1>
          <div className="flex flex-wrap items-center gap-2 mb-4">
            <Badge className="bg-recipe-primary hover:bg-recipe-primary/90">
              {recipe.strCategory}
            </Badge>
            <Badge variant="outline">{recipe.strArea} Cuisine</Badge>
            {recipe.strTags && recipe.strTags.split(',').map(tag => (
              <Badge key={tag} variant="secondary" className="bg-muted">
                {tag.trim()}
              </Badge>
            ))}
          </div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
          <div className="rounded-xl overflow-hidden">
            <img 
              src={recipe.strMealThumb} 
              alt={recipe.strMeal}
              className="w-full h-auto object-cover aspect-video"
              loading="eager" // Use eager loading for the main image above the fold
              fetchpriority="high"
            />
          </div>

          <div>
            <div className="flex space-x-4 mb-6">
              <Button
                variant={activeTab === "ingredients" ? "default" : "outline"}
                onClick={() => setActiveTab("ingredients")}
                className={activeTab === "ingredients" ? "bg-recipe-primary hover:bg-recipe-primary/90" : ""}
              >
                Ingredients
              </Button>
              <Button
                variant={activeTab === "instructions" ? "default" : "outline"}
                onClick={() => setActiveTab("instructions")}
                className={activeTab === "instructions" ? "bg-recipe-primary hover:bg-recipe-primary/90" : ""}
              >
                Instructions
              </Button>
            </div>

            {activeTab === "ingredients" ? (
              <div className="space-y-4">
                <h2 className="text-xl font-semibold">
                  Ingredients ({recipe.ingredients.length})
                </h2>
                <ul className="space-y-2">
                  {recipe.ingredients.map((item, index) => (
                    <li key={index} className="flex items-center py-1">
                      <span className="font-medium mr-2">•</span>
                      <span className="font-medium">{item.measure}</span>
                      <Separator orientation="vertical" className="mx-2 h-4" />
                      <span>{item.ingredient}</span>
                    </li>
                  ))}
                </ul>
              </div>
            ) : (
              <div className="space-y-4">
                <h2 className="text-xl font-semibold">Cooking Instructions</h2>
                {renderInstructions()}
              </div>
            )}
            {recipe.strYoutube && (
              <div className="mt-8">
                <h3 className="text-lg font-semibold mb-2">Video Tutorial</h3>
                <Button
                  variant="outline"
                  className="text-recipe-primary border-recipe-primary hover:bg-recipe-primary/10"
                  onClick={() => window.open(recipe.strYoutube, "_blank")}
                >
                  Watch on YouTube
                </Button>
              </div>
            )}
          </div>
        </div>
      </div>
    </>
  );
};

export default RecipeDetail;
```
> This code does the following:
- Displays a detailed view of a selected recipe.
- Uses structured schema.org JSON-LD for SEO.
- Implements a tab switcher to toggle between "Ingredients" and "Instructions".
- Splits cooking instructions into individual steps for better readability.
- Includes badges for category, cuisine, and tags.
- Uses eager loading with fetchpriority="high" for the main image.
- Provides a YouTube video button if available.

## ✅ SEO & Privacy Implementation

### 🔍 SEO Optimization

- **Meta Tags & Structured Data**:  
  Added basic meta tags (`<title>`, `<meta name="description">`) and [schema.org](https://schema.org/Recipe) structured data using `<script type="application/ld+json">` to improve search engine visibility and semantic understanding.

- **Performance & Core Web Vitals**:
  - Implemented **lazy loading** for images across recipe listings to improve load speed and reduce unused bytes.
  - Used `loading="eager"` and `fetchpriority="high"` only for above-the-fold images to ensure fast **Largest Contentful Paint (LCP)**.
  - Ensured optimized image display with proper `width`, `height`, and aspect ratios.

---

### 🛡️ Privacy Enhancements

- **No Tracking or Profiling**:
  - Our application does **not** use any form of tracking (e.g., Google Analytics, Facebook Pixel).
  - No cookies are used for profiling or behavioral analysis.
  - No third-party scripts are included that could compromise user anonymity.
  - As a result, users can interact with our platform without sacrificing their **privacy** or leaving digital footprints.

✅ With these measures, we ensure a **privacy-conscious**, performant, and search-engine-friendly experience.
