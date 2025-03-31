# Go Web Application Architecture

I'm migrating a TypeScript web application to Go using Gomponents and HTMX. Please help me port our TypeScript components following our established architecture patterns:

## 1. ARCHITECTURE REQUIREMENTS:
- Follow Go's package structure with clear separation between:
    * models/ - Data structures that act as a bridge between backend and UI
    * components/ - Reusable UI elements
    * pages/ - Full page compositions
    * handlers/ - Request handlers that connect to business logic
- Database/business logic calls only allowed in handlers
- Pass only UI models to components/pages

## 2. COMPONENT PATTERNS:
- Use strictly typed props structure for each component
- Implement conditional rendering with Iff() or If() functions
- Support both dark/light mode theming
- Follow composition pattern (small components composed into larger ones)
- Implement HTMX attributes for dynamic content and interactions

## 3. PAGE LAYOUT EXAMPLE:
```go
// pages/product_list.go
package pages

import (
  "fmt"
  . "maragu.dev/gomponents"
  . "maragu.dev/gomponents/components"
  . "maragu.dev/gomponents/html"
  "myapp/ui/components"
  "myapp/ui/models"
  "net/http"
)

// ProductListPageData contains all data needed to render the page
type ProductListPageData struct {
  Products    []models.Product
  Categories  []models.Category
  User        *models.User
  IsDarkMode  bool
  CurrentPage int
  TotalPages  int
}

// ProductListPage renders the complete product listing page
func ProductListPage(r *http.Request, data ProductListPageData) (Node, error) {
  // Get theme settings
  theme := components.GetTheme(r)
  colors := theme.Colors

  return HTML5(HTML5Props{
    Title: "Product Catalog | MyStore",
    Head: []Node{
      // Meta tags
      Meta(Name("description"), Content("Browse our product catalog")),
      Meta(Name("viewport"), Content("width=device-width, initial-scale=1")),
      
      // Scripts and styles
      Script(Src("https://unpkg.com/htmx.org@1.9.2")),
      Link(Rel("stylesheet"), Href("/static/css/tailwind.css")),
      
      // Dark mode initialization
      Raw(components.DarkModeScript()),
      
      // Additional component-specific JS
      Script(Src("/static/js/product-list.js"), Defer()),
    },
    Body: []Node{
      Div(
        Class(fmt.Sprintf("min-h-screen flex flex-col %s", colors["background"]["primary"])),
        
        // Navigation header
        components.NavBar(components.NavBarProps{
          CurrentUser: data.User,
          IsDarkMode: theme.IsDarkMode,
        }, colors),
        
        // Main content
        Main(
          Class("flex-grow container mx-auto px-4 py-8"),
          
          // Page header
          Div(
            Class("mb-8"),
            H1(
              Class(fmt.Sprintf("text-3xl font-bold %s", colors["text"]["primary"])),
              Text("Product Catalog"),
            ),
            P(
              Class(fmt.Sprintf("mt-2 %s", colors["text"]["secondary"])),
              Text("Browse our latest products"),
            ),
          ),
          
          // Two-column layout
          Div(
            Class("grid grid-cols-1 md:grid-cols-4 gap-6"),
            
            // Sidebar with filters
            Div(
              Class("md:col-span-1"),
              components.CategoryFilter(components.CategoryFilterProps{
                Categories: data.Categories,
                IsDarkMode: theme.IsDarkMode,
              }, colors),
            ),
            
            // Product grid
            Div(
              Class("md:col-span-3"),
              // Product grid container with HTMX for pagination
              Div(
                ID("product-grid"),
                Class("grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6"),
                // Map products to ProductCard components
                Group(func() []Node {
                  nodes := make([]Node, len(data.Products))
                  for i, product := range data.Products {
                    nodes[i] = components.ProductCard(components.ProductCardProps{
                      Product: product,
                      IsDarkMode: theme.IsDarkMode,
                      IsCompact: false,
                    }, colors)
                  }
                  return nodes
                }()),
              ),
              
              // Pagination
              components.Pagination(components.PaginationProps{
                CurrentPage: data.CurrentPage,
                TotalPages: data.TotalPages,
                IsDarkMode: theme.IsDarkMode,
              }, colors),
            ),
          ),
        ),
        
        // Footer
        components.Footer(components.FooterProps{
          IsDarkMode: theme.IsDarkMode,
        }, colors),
      ),
    },
  }), nil
}
```

## COMPONENT EXAMPLE:
```go
// components/product_card.go
package components

import (
  "fmt"
  . "maragu.dev/gomponents"
  . "maragu.dev/gomponents/html"
  hx "maragu.dev/gomponents-htmx"
  "myapp/ui/models"
)

type ProductCardProps struct {
  Product    models.Product
  IsDarkMode bool
  IsCompact  bool
}

func ProductCard(props ProductCardProps, colors ThemeMap) Node {
  return Div(
    Class(fmt.Sprintf("rounded-lg shadow-md %s %s",
      colors["background"]["card"],
      colors["border"]["default"])),
    
    // Product image
    Div(Class("relative pb-2/3 overflow-hidden"),
      Img(Src(props.Product.ImageURL), Alt(props.Product.Name)),
    ),
    
    // Product details
    Div(Class("p-4"),
      H3(
        Class(fmt.Sprintf("text-lg font-semibold %s", colors["text"]["primary"])),
        Text(props.Product.Name),
      ),
      Iff(!props.IsCompact, func() Node {
        return P(
          Class(fmt.Sprintf("mt-2 %s", colors["text"]["secondary"])), 
          Text(props.Product.Description),
        )
      }),
      Div(
        Class("mt-4 flex justify-between items-center"),
        Span(
          Class(fmt.Sprintf("font-bold %s", colors["text"]["primary"])),
          Text(fmt.Sprintf("$%.2f", props.Product.Price)),
        ),
        Button(
          hx.Post(fmt.Sprintf("/api/cart/add/%d", props.Product.ID)),
          hx.Target("#cart-count"),
          hx.Swap("innerHTML"),
          Class(fmt.Sprintf("px-3 py-1 rounded-md %s", colors["button"]["primary"])),
          Text("Add to Cart"),
        ),
      ),
    ),
  )
}
```

## MODEL EXAMPLE:
```go
// models/models.go
package models

type User struct {
  ID        int
  Username  string
  Email     string
  Token     string
  AvatarURL string
}

type Product struct {
  ID          int
  Name        string
  Description string
  Price       float64
  ImageURL    string
  Category    string
  InStock     bool
}

type Category struct {
  ID    int
  Name  string
  Slug  string
  Count int
}
```

## HANDLER EXAMPLE:
```go
// handlers/products.go
package handlers

import (
  "fmt"
  "net/http"
  "strconv"
  "your-app/db"
  "your-app/internal/products"
  "your-app/internal/users"
  "your-app/ui/models"
  "your-app/ui/pages"
)

// ProductListHandler handles requests for the product listing page
func ProductListHandler(dbClient *db.Client) func(w http.ResponseWriter, r *http.Request) {
  return func(w http.ResponseWriter, r *http.Request) {
    // Get pagination parameters
    page, _ := strconv.Atoi(r.URL.Query().Get("page"))
    if page < 1 {
      page = 1
    }
    
    // Get optional category filter
    categorySlug := r.URL.Query().Get("category")
    
    // Get theme (light/dark mode) from request
    isDarkMode := isDarkModeEnabled(r)
    
    // Get current user from request
    currentUser, err := users.GetUserFromRequest(r.Context(), dbClient, r)
    var uiUser *models.User
    if err == nil && currentUser != nil {
      uiUser = &models.User{
        ID:        currentUser.ID,
        Username:  currentUser.Username,
        Email:     currentUser.Email,
        Token:     currentUser.FirebaseID,
        AvatarURL: currentUser.AvatarURL,
      }
    }
    
    // Get products from database
    dbProducts, totalProducts, err := products.GetProducts(r.Context(), dbClient, page, 12, categorySlug)
    if err != nil {
      http.Error(w, "Failed to fetch products", http.StatusInternalServerError)
      return
    }
    
    // Get categories from database
    dbCategories, err := products.GetCategories(r.Context(), dbClient)
    if err != nil {
      http.Error(w, "Failed to fetch categories", http.StatusInternalServerError)
      return
    }
    
    // Convert to UI models
    uiProducts := make([]models.Product, len(dbProducts))
    for i, p := range dbProducts {
      uiProducts[i] = models.Product{
        ID:          p.ID,
        Name:        p.Name,
        Description: p.Description,
        Price:       p.Price,
        ImageURL:    p.ImageURL,
        Category:    p.Category,
        InStock:     p.StockCount > 0,
      }
    }
    
    uiCategories := make([]models.Category, len(dbCategories))
    for i, c := range dbCategories {
      uiCategories[i] = models.Category{
        ID:    c.ID,
        Name:  c.Name,
        Slug:  c.Slug,
        Count: c.ProductCount,
      }
    }
    
    // Calculate pagination
    totalPages := (totalProducts + 11) / 12 // ceiling division
    
    // Prepare page data
    pageData := pages.ProductListPageData{
      Products:    uiProducts,
      Categories:  uiCategories,
      User:        uiUser,
      IsDarkMode:  isDarkMode,
      CurrentPage: page,
      TotalPages:  totalPages,
    }
    
    // Render the page
    node, err := pages.ProductListPage(r, pageData)
    if err != nil {
      http.Error(w, fmt.Sprintf("Failed to render page: %v", err), http.StatusInternalServerError)
      return
    }
    
    w.Header().Set("Content-Type", "text/html; charset=utf-8")
    w.WriteHeader(http.StatusOK)
    err = node.Render(w)
    if err != nil {
      return
    }
  }
}

// Helper function to check dark mode preference
func isDarkModeEnabled(r *http.Request) bool {
  cookie, err := r.Cookie("darkMode")
  if err == nil {
    return cookie.Value == "true"
  }
  return false
}
```

Please port the following TypeScript component to Go using our patterns above:
[INSERT TYPESCRIPT COMPONENT HERE]
Please return:

The complete Go implementation of the component
A modified version of our models/models.go to support this component
A simple example of a handler function that would provide data to this component